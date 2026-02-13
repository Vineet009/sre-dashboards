# GCP Monitoring Data Collection Strategy for Multi-Level Reliability Dashboard

## Overview
This document outlines the meaningful metrics to fetch from Google Cloud Monitoring (formerly Stackdriver) for a 3-level reliability dashboard covering Department → Application → CUJ → SLO hierarchy.

## GCP Projects Structure
- Each application deployed in separate GCP project
- 10 applications total across department
- Need to aggregate data across projects for department-level view

---

## Level 1: Department View (Executive/Department Head)

### Key Metrics to Fetch

#### 1. **Overall Reliability Score**
Aggregate across all 10 GCP projects:

```python
from google.cloud import monitoring_v3
from google.cloud.monitoring_dashboard import v1

# Metrics to fetch per project:
metrics = [
    'compute.googleapis.com/instance/uptime',
    'run.googleapis.com/request_count',
    'run.googleapis.com/request_latencies',
    'loadbalancing.googleapis.com/https/request_count',
    'loadbalancing.googleapis.com/https/backend_latencies',
]

# Key Queries:
# 1. Service Availability (%)
metric_type = 'run.googleapis.com/container/billable_instance_time'
# Calculate uptime percentage

# 2. Error Rate
metric_type = 'run.googleapis.com/request_count'
# Filter: metric.response_code_class="5xx" OR "4xx"
# Formula: (error_requests / total_requests) * 100

# 3. Latency Score
metric_type = 'loadbalancing.googleapis.com/https/backend_latencies'
# Percentile: 95th, 99th
```

#### 2. **Aggregated Availability (99.2%)**
```python
# For each project:
availability_metrics = {
    'compute_engine': 'compute.googleapis.com/instance/uptime',
    'cloud_run': 'run.googleapis.com/container/billable_instance_time',
    'gke': 'container.googleapis.com/container/uptime',
    'cloud_functions': 'cloudfunctions.googleapis.com/function/execution_count',
    'app_engine': 'appengine.googleapis.com/http/server/response_count'
}

# Calculate: (total_uptime / total_time) * 100
# Aggregate across all 10 projects
```

#### 3. **Aggregated Latency Score (97.8%)**
```python
latency_metrics = {
    'load_balancer': 'loadbalancing.googleapis.com/https/backend_latencies',
    'cloud_run': 'run.googleapis.com/request_latencies',
    'app_engine': 'appengine.googleapis.com/http/server/response_latencies',
    'api_gateway': 'apigateway.googleapis.com/api/request_latencies'
}

# Calculate P95, P99 latencies
# Compare against SLO targets
# Score = (requests_meeting_slo / total_requests) * 100
```

#### 4. **6-Month Trend Data**
```python
# Time range: Last 6 months
# Granularity: Daily or Weekly aggregates
# For each metric: reliability, availability, latency

time_series_params = {
    'interval': {
        'end_time': now,
        'start_time': now - 180 days  # 6 months
    },
    'aggregation': {
        'alignment_period': {'seconds': 86400},  # Daily
        'per_series_aligner': 'ALIGN_MEAN',
        'cross_series_reducer': 'REDUCE_MEAN'
    }
}
```

### Data Points for Department View
```json
{
  "department_name": "Engineering Department",
  "total_applications": 10,
  "metrics": {
    "overall_reliability": {
      "current": 98.7,
      "trend_6_months": [98.5, 98.6, 98.8, 98.7, 98.9, 98.7],
      "months": ["Aug", "Sep", "Oct", "Nov", "Dec", "Jan"]
    },
    "overall_availability": {
      "current": 99.2,
      "trend_6_months": [99.1, 99.2, 99.3, 99.2, 99.1, 99.2]
    },
    "overall_latency_score": {
      "current": 97.8,
      "trend_6_months": [97.5, 97.7, 97.9, 97.8, 98.0, 97.8]
    }
  },
  "actionable_insights": [
    {
      "priority": "high",
      "application": "Inventory System",
      "issue": "Latency degradation detected",
      "recommendation": "Review Cloud SQL query performance"
    }
  ]
}
```

---

## Level 2: Application View (Product Owner)

### Metrics per GCP Project

#### 1. **Application-Level Metrics**

```python
# For a single GCP project (e.g., payment-service-prod):

PROJECT_ID = 'payment-service-prod'

# Core Service Metrics
application_metrics = {
    # Compute/Container Health
    'instance_count': 'compute.googleapis.com/instance/count',
    'cpu_utilization': 'compute.googleapis.com/instance/cpu/utilization',
    'memory_utilization': 'compute.googleapis.com/instance/memory/utilization',
    
    # Request Metrics
    'request_count': 'run.googleapis.com/request_count',
    'request_latencies': 'run.googleapis.com/request_latencies',
    
    # Error Tracking
    'error_count': 'logging.googleapis.com/log_entry_count',  # filter: severity>=ERROR
    
    # Database (Cloud SQL)
    'db_connections': 'cloudsql.googleapis.com/database/mysql/connections',
    'db_query_latency': 'cloudsql.googleapis.com/database/mysql/queries',
    
    # Pub/Sub (if used)
    'pubsub_publish_latency': 'pubsub.googleapis.com/topic/send_request_latencies',
    'pubsub_dead_letter': 'pubsub.googleapis.com/subscription/dead_letter_message_count'
}
```

#### 2. **CUJ-Specific Metrics** (Critical User Journeys)

```python
# CUJ: Payment Processing
# Define by URL path, service endpoint, or trace name

cuj_payment_processing = {
    'name': 'Payment Processing',
    
    # Filter requests by specific path
    'filter': 'metric.type="run.googleapis.com/request_count" AND resource.label.service_name="payment-api" AND metric.label.path="/api/v1/process-payment"',
    
    'metrics': {
        # Request Success Rate
        'success_rate': {
            'metric': 'run.googleapis.com/request_count',
            'filter': 'metric.response_code_class!="5xx"',
            'calculation': '(successful / total) * 100'
        },
        
        # Latency Distribution
        'p50_latency': 'run.googleapis.com/request_latencies',
        'p95_latency': 'run.googleapis.com/request_latencies',
        'p99_latency': 'run.googleapis.com/request_latencies',
        
        # Throughput
        'requests_per_minute': 'run.googleapis.com/request_count',
        
        # Error Budget
        'error_budget_remaining': {
            'slo_target': 99.9,  # 99.9% availability
            'current_availability': 99.6,
            'budget_consumed': '(100 - 99.6) / (100 - 99.9) = 40%',
            'budget_remaining': '60%'
        }
    }
}

# CUJ: Refund Workflow
cuj_refund = {
    'name': 'Refund Workflow',
    'filter': 'resource.label.service_name="refund-processor"',
    'metrics': {
        'queue_depth': 'pubsub.googleapis.com/subscription/num_undelivered_messages',
        'processing_time': 'run.googleapis.com/request_latencies',
        'success_rate': 'calculated from request_count with filters'
    }
}
```

#### 3. **Application Health Status**

```python
# Determine health status based on SLO compliance
def calculate_health_status(metrics):
    """
    Returns: 'healthy', 'warning', 'critical'
    """
    reliability = metrics['reliability']
    availability = metrics['availability']
    latency_score = metrics['latency_score']
    
    if all([reliability >= 99, availability >= 99.5, latency_score >= 98]):
        return 'healthy'
    elif all([reliability >= 97, availability >= 98, latency_score >= 95]):
        return 'warning'
    else:
        return 'critical'
```

### Data Points for Application View

```json
{
  "application": {
    "name": "Payment Service",
    "gcp_project_id": "payment-service-prod",
    "status": "healthy",
    "metrics": {
      "reliability": 99.1,
      "availability": 99.5,
      "latency_score": 98.2,
      "total_requests_24h": 1250000,
      "error_rate": 0.09,
      "avg_cpu_utilization": 45.2,
      "avg_memory_utilization": 62.1
    },
    "trends_6_months": { /* same structure as department */ },
    "cujs": [
      {
        "name": "Payment Processing",
        "endpoint": "/api/v1/process-payment",
        "reliability": 99.3,
        "availability": 99.6,
        "latency_score": 98.5,
        "request_count_24h": 850000,
        "p95_latency_ms": 145,
        "p99_latency_ms": 230
      }
    ]
  }
}
```

---

## Level 3: SLO View (Dev & Ops Teams)

### Detailed SLO Metrics

#### 1. **Latency SLOs**

```python
# SLO: API Latency < 200ms (95th percentile)
latency_slo = {
    'name': 'API Latency',
    'metric': 'run.googleapis.com/request_latencies',
    'threshold': 200,  # ms
    'percentile': 95,
    'target': 95,  # 95% of requests must meet threshold
    
    'query': """
    fetch cloud_run_revision
    | metric 'run.googleapis.com/request_latencies'
    | filter resource.service_name == 'payment-api'
    | align delta(1m)
    | every 1m
    | group_by [resource.service_name],
        [value_latency_percentile: percentile(value.request_latencies, 95)]
    """,
    
    'current_value': 185,  # ms (P95)
    'compliance': 98.5,  # % of time windows meeting SLO
    'error_budget': {
        'total': 5.0,  # 100 - 95 = 5% allowed error
        'consumed': 1.5,  # % already consumed
        'remaining': 3.5,  # % remaining
        'remaining_percentage': 70  # (3.5 / 5.0) * 100
    }
}
```

#### 2. **Availability SLOs**

```python
# SLO: Service Uptime 99%
availability_slo = {
    'name': 'Service Uptime',
    'metric_type': 'monitoring.googleapis.com/uptime_check/check_passed',
    'target': 99.0,
    
    'query': """
    fetch uptime_url
    | metric 'monitoring.googleapis.com/uptime_check/check_passed'
    | filter resource.project_id == 'payment-service-prod'
    | align rate(1m)
    | every 1m
    | group_by [resource.host],
        [value_check_passed_mean: mean(value.check_passed)]
    """,
    
    'current_value': 99.6,
    'uptime_percentage': 99.6,
    'downtime_minutes_30d': 172.8,  # (100 - 99.6) * 43200 minutes
    'error_budget': {
        'total_allowed_downtime_30d': 432,  # 1% of 30 days
        'consumed_downtime_30d': 172.8,
        'remaining_downtime_30d': 259.2,
        'remaining_percentage': 60
    }
}
```

#### 3. **Error Rate SLOs**

```python
# SLO: Error Rate < 1%
error_rate_slo = {
    'name': 'Error Rate',
    'target': 99.0,  # Success rate target (error rate < 1%)
    
    'queries': {
        'total_requests': """
        fetch cloud_run_revision
        | metric 'run.googleapis.com/request_count'
        | filter resource.service_name == 'payment-api'
        | align rate(1m)
        | every 1m
        | group_by [], [value_request_count_sum: sum(value.request_count)]
        """,
        
        'error_requests': """
        fetch cloud_run_revision
        | metric 'run.googleapis.com/request_count'
        | filter resource.service_name == 'payment-api'
        | filter metric.response_code_class == '5xx' || metric.response_code_class == '4xx'
        | align rate(1m)
        | every 1m
        | group_by [], [value_error_count_sum: sum(value.request_count)]
        """
    },
    
    'calculation': {
        'success_rate': '((total - errors) / total) * 100',
        'error_rate': '(errors / total) * 100'
    },
    
    'current_metrics': {
        'total_requests_24h': 850000,
        'error_requests_24h': 5100,
        'error_rate': 0.6,
        'success_rate': 99.4,
        'compliance': 99.4  # meeting target of 99%
    },
    
    'error_budget': {
        'allowed_error_rate': 1.0,
        'actual_error_rate': 0.6,
        'consumed_percentage': 60,
        'remaining_percentage': 40
    }
}
```

#### 4. **Custom Business Metrics**

```python
# Transaction-specific metrics from Cloud Logging
transaction_slos = {
    'payment_success_rate': {
        'metric': 'logging.googleapis.com/user/payment_transaction_result',
        'filter': 'jsonPayload.transaction_type="payment" AND jsonPayload.result="success"',
        'target': 99.5,
        'current': 99.6
    },
    
    'refund_processing_time': {
        'metric': 'logging.googleapis.com/user/refund_duration',
        'filter': 'jsonPayload.process_type="refund"',
        'threshold': 500,  # ms
        'target': 95,  # 95% under 500ms
        'current_p95': 478
    }
}
```

#### 5. **Infrastructure SLOs**

```python
infrastructure_slos = {
    # Database Performance
    'db_connection_pool': {
        'metric': 'cloudsql.googleapis.com/database/mysql/connections',
        'threshold': 80,  # max connections
        'alert_threshold': 70,  # 85% capacity
        'current': 45
    },
    
    'db_query_latency': {
        'metric': 'cloudsql.googleapis.com/database/mysql/queries',
        'threshold_p99': 100,  # ms
        'current_p99': 85
    },
    
    # Cache Hit Rate
    'memorystore_hit_rate': {
        'metric': 'redis.googleapis.com/stats/cache_hit_ratio',
        'target': 95,
        'current': 97.2
    },
    
    # Message Queue
    'pubsub_oldest_unacked_age': {
        'metric': 'pubsub.googleapis.com/subscription/oldest_unacked_message_age',
        'threshold': 300,  # seconds
        'current': 12
    }
}
```

### Data Points for SLO View

```json
{
  "cuj": {
    "name": "Payment Processing",
    "application": "Payment Service",
    "description": "End-to-end payment transaction flow",
    "slos": [
      {
        "id": "slo-latency-api",
        "name": "API Response Time",
        "type": "latency",
        "metric_type": "run.googleapis.com/request_latencies",
        "target": "<200ms (P95)",
        "threshold": 95,
        "current_value": 98.5,
        "actual_p95_ms": 185,
        "error_budget": {
          "total_percentage": 5.0,
          "consumed_percentage": 1.5,
          "remaining_percentage": 3.5,
          "remaining_display": 70,
          "status": "healthy"
        },
        "last_24h": {
          "total_requests": 850000,
          "requests_meeting_slo": 837250,
          "compliance_rate": 98.5
        },
        "trend_7d": [98.2, 98.4, 98.6, 98.5, 98.7, 98.5, 98.5],
        "actionable_items": [
          {
            "priority": "medium",
            "action": "Monitor query performance on user_payments table",
            "reason": "Slight increase in P99 latency detected"
          }
        ]
      },
      {
        "id": "slo-availability-service",
        "name": "Service Availability",
        "type": "availability",
        "metric_type": "monitoring.googleapis.com/uptime_check/check_passed",
        "target": "99.0%",
        "threshold": 99,
        "current_value": 99.6,
        "error_budget": {
          "total_downtime_allowed_30d_minutes": 432,
          "consumed_downtime_30d_minutes": 172.8,
          "remaining_downtime_30d_minutes": 259.2,
          "remaining_display": 60,
          "status": "healthy"
        },
        "incidents_30d": [
          {
            "date": "2025-01-05",
            "duration_minutes": 45,
            "impact": "Partial outage - EU region",
            "root_cause": "Load balancer configuration"
          }
        ]
      },
      {
        "id": "slo-errors-rate",
        "name": "Error Rate",
        "type": "reliability",
        "metric_type": "run.googleapis.com/request_count",
        "target": "<1% error rate",
        "threshold": 99,
        "current_value": 99.4,
        "error_rate": 0.6,
        "error_budget": {
          "allowed_error_rate": 1.0,
          "actual_error_rate": 0.6,
          "consumed_percentage": 60,
          "remaining_display": 40,
          "status": "healthy"
        },
        "error_breakdown_24h": {
          "4xx_errors": 3400,
          "5xx_errors": 1700,
          "total_errors": 5100,
          "top_errors": [
            {"code": "401", "count": 2100, "message": "Unauthorized"},
            {"code": "503", "count": 1200, "message": "Service Unavailable"},
            {"code": "500", "count": 500, "message": "Internal Server Error"}
          ]
        }
      }
    ]
  }
}
```

---

## Python Script to Fetch All Data

```python
#!/usr/bin/env python3
"""
GCP Monitoring Data Fetcher for Reliability Dashboard
Fetches metrics from multiple GCP projects and aggregates them
"""

from google.cloud import monitoring_v3
from google.cloud import logging_v2
from datetime import datetime, timedelta
import json
from typing import List, Dict

class GCPReliabilityDataFetcher:
    def __init__(self, project_ids: List[str]):
        self.project_ids = project_ids
        self.client = monitoring_v3.MetricServiceClient()
        
    def fetch_time_series(self, project_id: str, metric_type: str, 
                         hours_ago: int = 24, filters: str = None):
        """Fetch time series data for a specific metric"""
        project_name = f"projects/{project_id}"
        
        now = datetime.utcnow()
        interval = monitoring_v3.TimeInterval({
            "end_time": now,
            "start_time": now - timedelta(hours=hours_ago),
        })
        
        filter_str = f'metric.type = "{metric_type}"'
        if filters:
            filter_str += f' AND {filters}'
        
        request = monitoring_v3.ListTimeSeriesRequest({
            "name": project_name,
            "filter": filter_str,
            "interval": interval,
            "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
        })
        
        results = self.client.list_time_series(request=request)
        return list(results)
    
    def calculate_availability(self, project_id: str) -> float:
        """Calculate availability percentage"""
        # Fetch uptime check data
        time_series = self.fetch_time_series(
            project_id,
            'monitoring.googleapis.com/uptime_check/check_passed',
            hours_ago=24
        )
        
        if not time_series:
            return 100.0
        
        total_checks = 0
        passed_checks = 0
        
        for series in time_series:
            for point in series.points:
                total_checks += 1
                if point.value.bool_value or point.value.double_value > 0.5:
                    passed_checks += 1
        
        if total_checks == 0:
            return 100.0
            
        return (passed_checks / total_checks) * 100
    
    def calculate_error_rate(self, project_id: str, service_name: str = None) -> Dict:
        """Calculate error rate from request counts"""
        filter_str = None
        if service_name:
            filter_str = f'resource.service_name = "{service_name}"'
        
        # Total requests
        total_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_count',
            hours_ago=24,
            filters=filter_str
        )
        
        # Error requests (4xx and 5xx)
        error_filter = f'metric.response_code_class = "5xx" OR metric.response_code_class = "4xx"'
        if filter_str:
            error_filter = f'{filter_str} AND ({error_filter})'
        
        error_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_count',
            hours_ago=24,
            filters=error_filter
        )
        
        total_requests = sum(
            sum(point.value.int64_value or point.value.double_value 
                for point in series.points)
            for series in total_series
        )
        
        error_requests = sum(
            sum(point.value.int64_value or point.value.double_value 
                for point in series.points)
            for series in error_series
        )
        
        error_rate = (error_requests / total_requests * 100) if total_requests > 0 else 0
        success_rate = 100 - error_rate
        
        return {
            'total_requests': total_requests,
            'error_requests': error_requests,
            'error_rate': error_rate,
            'success_rate': success_rate
        }
    
    def calculate_latency_score(self, project_id: str, 
                               threshold_ms: int = 200,
                               target_percentile: float = 95) -> Dict:
        """Calculate latency compliance score"""
        time_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_latencies',
            hours_ago=24
        )
        
        latencies = []
        for series in time_series:
            for point in series.points:
                # Distribution values
                if hasattr(point.value, 'distribution_value'):
                    dist = point.value.distribution_value
                    # Extract percentiles if available
                    latencies.append(dist.mean)
        
        if not latencies:
            return {'score': 100, 'p95': 0, 'p99': 0}
        
        latencies.sort()
        p95_index = int(len(latencies) * 0.95)
        p99_index = int(len(latencies) * 0.99)
        
        p95_latency = latencies[p95_index] if p95_index < len(latencies) else latencies[-1]
        p99_latency = latencies[p99_index] if p99_index < len(latencies) else latencies[-1]
        
        # Calculate score based on threshold
        requests_meeting_slo = sum(1 for lat in latencies if lat <= threshold_ms)
        score = (requests_meeting_slo / len(latencies)) * 100
        
        return {
            'score': score,
            'p50': latencies[len(latencies)//2],
            'p95': p95_latency,
            'p99': p99_latency,
            'threshold_ms': threshold_ms
        }
    
    def fetch_department_data(self) -> Dict:
        """Aggregate data across all projects for department view"""
        applications = []
        total_reliability = 0
        total_availability = 0
        total_latency_score = 0
        
        for project_id in self.project_ids:
            # Fetch per-project metrics
            availability = self.calculate_availability(project_id)
            error_data = self.calculate_error_rate(project_id)
            latency_data = self.calculate_latency_score(project_id)
            
            reliability = error_data['success_rate']
            latency_score = latency_data['score']
            
            total_reliability += reliability
            total_availability += availability
            total_latency_score += latency_score
            
            applications.append({
                'project_id': project_id,
                'reliability': reliability,
                'availability': availability,
                'latency_score': latency_score
            })
        
        num_apps = len(self.project_ids)
        
        return {
            'department': {
                'name': 'Engineering Department',
                'total_applications': num_apps,
                'overall_reliability': total_reliability / num_apps,
                'overall_availability': total_availability / num_apps,
                'overall_latency_score': total_latency_score / num_apps,
            },
            'applications': applications
        }
    
    def fetch_application_data(self, project_id: str) -> Dict:
        """Fetch detailed metrics for a single application"""
        availability = self.calculate_availability(project_id)
        error_data = self.calculate_error_rate(project_id)
        latency_data = self.calculate_latency_score(project_id)
        
        return {
            'project_id': project_id,
            'reliability': error_data['success_rate'],
            'availability': availability,
            'latency_score': latency_data['score'],
            'total_requests_24h': error_data['total_requests'],
            'error_rate': error_data['error_rate'],
            'p95_latency_ms': latency_data['p95'],
            'p99_latency_ms': latency_data['p99']
        }


# Usage Example
if __name__ == '__main__':
    # List of all GCP project IDs (one per application)
    project_ids = [
        'payment-service-prod',
        'auth-service-prod',
        'inventory-service-prod',
        'analytics-pipeline-prod',
        # ... add remaining 6 projects
    ]
    
    fetcher = GCPReliabilityDataFetcher(project_ids)
    
    # Fetch department-level data
    dept_data = fetcher.fetch_department_data()
    print(json.dumps(dept_data, indent=2))
    
    # Fetch application-level data
    app_data = fetcher.fetch_application_data('payment-service-prod')
    print(json.dumps(app_data, indent=2))
```

---

## Summary: Key GCP Metrics by Level

### Level 1 - Department (10 Projects Aggregated)
- `monitoring.googleapis.com/uptime_check/check_passed` - Overall availability
- `run.googleapis.com/request_count` (filtered by error codes) - Error rate
- `run.googleapis.com/request_latencies` - Latency percentiles
- Aggregate across all 10 projects

### Level 2 - Application (Single Project)
- Same metrics as Level 1 but for single project
- `compute.googleapis.com/instance/cpu/utilization` - Resource usage
- `cloudsql.googleapis.com/database/*/connections` - Database health
- Filter by service name for CUJ-level granularity

### Level 3 - SLO (Single CUJ)
- `run.googleapis.com/request_latencies` with percentile calculations
- `monitoring.googleapis.com/uptime_check/*` for availability
- Custom log-based metrics for business transactions
- Error budget calculations based on SLO targets

This structure provides actionable, role-specific insights at each organizational level.
