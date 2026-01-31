# Monitoring Architecture

## Overview Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE MONITOR (Platform)                             │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                   LOG ANALYTICS WORKSPACE                           │     │
│  │                    (Central Data Store)                             │     │
│  │                                                                     │     │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │     │
│  │  │ ContainerApp    │  │ ContainerApp    │  │ App Insights    │    │     │
│  │  │ ConsoleLogs_CL  │  │ SystemLogs_CL   │  │ Tables          │    │     │
│  │  │ (your logs)     │  │ (ACA events)    │  │ (traces, reqs)  │    │     │
│  │  └────────▲────────┘  └────────▲────────┘  └────────▲────────┘    │     │
│  └───────────│────────────────────│────────────────────│─────────────┘     │
│              │                    │                    │                    │
│              │ automatic          │ automatic          │ SDK                │
│              │                    │                    │                    │
│  ┌───────────┴────────────────────┴────────────────────┴─────────────┐     │
│  │              CONTAINER APPS ENVIRONMENT                            │     │
│  │                                                                    │     │
│  │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │     │
│  │   │ user    │ │ product │ │ order   │ │ payment │ │  ...    │   │     │
│  │   │ service │ │ service │ │ service │ │ service │ │         │   │     │
│  │   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │ APPLICATION      │  │ ALERTS           │  │ DASHBOARDS       │          │
│  │ INSIGHTS         │  │ (notifications)  │  │ (visualizations) │          │
│  │ (APM + tracing)  │  │                  │  │                  │          │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘          │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Components

| Component                      | Purpose                                                  |
| ------------------------------ | -------------------------------------------------------- |
| **Azure Monitor**              | Umbrella platform for all monitoring                     |
| **Log Analytics Workspace**    | Central database for all logs                            |
| **Application Insights**       | APM: distributed tracing, request tracking, dependencies |
| **Container Apps Environment** | Auto-sends stdout/stderr to Log Analytics                |

## Data Flows

| Flow          | Source                           | Destination                          | How          |
| ------------- | -------------------------------- | ------------------------------------ | ------------ |
| Console Logs  | `console.log()`, `print()`, etc. | `ContainerAppConsoleLogs_CL`         | Automatic    |
| System Logs   | ACA platform events              | `ContainerAppSystemLogs_CL`          | Automatic    |
| App Telemetry | App Insights SDK                 | `traces`, `requests`, `dependencies` | Requires SDK |

## Quick Queries (Log Analytics)

**All errors across services:**

```kusto
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(1h)
| where Log_s contains "error" or Log_s contains "exception"
| project TimeGenerated, Service=ContainerAppName_s, Message=Log_s
| order by TimeGenerated desc
```

**Error count by service:**

```kusto
ContainerAppConsoleLogs_CL
| where TimeGenerated > ago(1h)
| where Log_s contains "error"
| summarize Errors=count() by Service=ContainerAppName_s
| order by Errors desc
```

**System events (restarts, crashes):**

```kusto
ContainerAppSystemLogs_CL
| where TimeGenerated > ago(1h)
| project TimeGenerated, Service=ContainerAppName_s, Event=Log_s
```

## Portal Access

- **Log Analytics** → Logs → Run KQL queries
- **Application Insights** → Application Map → Visual service dependencies
- **Application Insights** → Failures → Error breakdown
- **Application Insights** → Performance → Latency analysis
