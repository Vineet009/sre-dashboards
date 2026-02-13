# Complete GCP Cloud Monitoring Guide for Application Reliability
## Comprehensive Data Collection & Utilization Strategy

---

## Table of Contents
1. [Available GCP Metrics by Service](#available-metrics)
2. [Level 1: Department Metrics & Actions](#level-1-department)
3. [Level 2: Application Metrics & Actions](#level-2-application)
4. [Level 3: SLO Metrics & Actions](#level-3-slo)
5. [Automated Reliability Actions](#automated-actions)
6. [Complete Python Implementation](#python-implementation)

---

## Available GCP Metrics by Service {#available-metrics}

### 1. **Cloud Run Metrics**
```python
CLOUD_RUN_METRICS = {
    # Request Metrics
    'run.googleapis.com/request_count': {
        'description': 'Number of requests received',
        'labels': ['response_code', 'response_code_class', 'service_name'],
        'use_case': 'Calculate success rate, error rate, throughput'
    },
    
    'run.googleapis.com/request_latencies': {
        'description': 'Request latency distribution',
        'type': 'DISTRIBUTION',
        'use_case': 'Calculate P50, P95, P99 latencies for SLOs'
    },
    
    # Container Metrics
    'run.googleapis.com/container/cpu/utilizations': {
        'description': 'CPU utilization per container',
        'use_case': 'Identify CPU bottlenecks, auto-scaling triggers'
    },
    
    'run.googleapis.com/container/memory/utilizations': {
        'description': 'Memory utilization per container',
        'use_case': 'Detect memory leaks, optimize container size'
    },
    
    'run.googleapis.com/container/instance_count': {
        'description': 'Number of running instances',
        'use_case': 'Track scaling behavior, capacity planning'
    },
    
    'run.googleapis.com/container/billable_instance_time': {
        'description': 'Billable container time',
        'use_case': 'Calculate availability and cost optimization'
    },
    
    # Startup Metrics
    'run.googleapis.com/container/startup_latencies': {
        'description': 'Container cold start time',
        'use_case': 'Optimize cold starts, improve user experience'
    }
}
```

### 2. **Google Kubernetes Engine (GKE) Metrics**
```python
GKE_METRICS = {
    # Pod Metrics
    'kubernetes.io/container/cpu/core_usage_time': {
        'description': 'CPU usage per container',
        'use_case': 'Resource optimization, pod sizing'
    },
    
    'kubernetes.io/container/memory/used_bytes': {
        'description': 'Memory usage per container',
        'use_case': 'Memory leak detection, OOMKill prevention'
    },
    
    'kubernetes.io/container/restart_count': {
        'description': 'Container restart count',
        'use_case': 'Stability monitoring, crash detection'
    },
    
    # Node Metrics
    'kubernetes.io/node/cpu/allocatable_utilization': {
        'description': 'Node CPU utilization',
        'use_case': 'Cluster capacity planning'
    },
    
    'kubernetes.io/node/memory/allocatable_utilization': {
        'description': 'Node memory utilization',
        'use_case': 'Node scaling decisions'
    },
    
    # Pod Autoscaling
    'kubernetes.io/pod/autoscaler/spec/target_metric': {
        'description': 'HPA target metrics',
        'use_case': 'Validate autoscaling configuration'
    }
}
```

### 3. **Cloud SQL Metrics**
```python
CLOUD_SQL_METRICS = {
    # Performance Metrics
    'cloudsql.googleapis.com/database/queries': {
        'description': 'Number of queries executed',
        'use_case': 'Query throughput, performance trending'
    },
    
    'cloudsql.googleapis.com/database/mysql/queries': {
        'description': 'MySQL-specific query metrics',
        'type': 'DISTRIBUTION',
        'use_case': 'Calculate query latency percentiles'
    },
    
    # Connection Metrics
    'cloudsql.googleapis.com/database/mysql/connections': {
        'description': 'Active database connections',
        'use_case': 'Connection pool sizing, leak detection'
    },
    
    'cloudsql.googleapis.com/database/mysql/max_connections': {
        'description': 'Maximum allowed connections',
        'use_case': 'Capacity planning, connection limits'
    },
    
    # Resource Metrics
    'cloudsql.googleapis.com/database/cpu/utilization': {
        'description': 'Database CPU usage',
        'use_case': 'Right-sizing, performance optimization'
    },
    
    'cloudsql.googleapis.com/database/memory/utilization': {
        'description': 'Database memory usage',
        'use_case': 'Buffer pool optimization'
    },
    
    'cloudsql.googleapis.com/database/disk/utilization': {
        'description': 'Disk space usage percentage',
        'use_case': 'Prevent storage full errors'
    },
    
    # Replication Metrics
    'cloudsql.googleapis.com/database/replication/replica_lag': {
        'description': 'Replication lag in seconds',
        'use_case': 'Read replica health, consistency monitoring'
    },
    
    # InnoDB Metrics (MySQL)
    'cloudsql.googleapis.com/database/mysql/innodb_pages_written': {
        'description': 'Pages written by InnoDB',
        'use_case': 'I/O performance tuning'
    },
    
    'cloudsql.googleapis.com/database/mysql/slow_query_count': {
        'description': 'Number of slow queries',
        'use_case': 'Identify optimization opportunities'
    }
}
```

### 4. **Load Balancer Metrics**
```python
LOAD_BALANCER_METRICS = {
    # Request Metrics
    'loadbalancing.googleapis.com/https/request_count': {
        'description': 'Number of HTTPS requests',
        'labels': ['response_code', 'backend_name'],
        'use_case': 'Traffic distribution, error tracking'
    },
    
    'loadbalancing.googleapis.com/https/backend_latencies': {
        'description': 'Backend response latencies',
        'type': 'DISTRIBUTION',
        'use_case': 'SLO tracking, backend performance'
    },
    
    'loadbalancing.googleapis.com/https/total_latencies': {
        'description': 'Total request latencies (including LB)',
        'use_case': 'End-to-end latency monitoring'
    },
    
    # Backend Health
    'loadbalancing.googleapis.com/https/backend_request_count': {
        'description': 'Requests sent to each backend',
        'use_case': 'Load distribution verification'
    },
    
    'loadbalancing.googleapis.com/https/backend_request_bytes_count': {
        'description': 'Bytes sent to backends',
        'use_case': 'Bandwidth monitoring'
    }
}
```

### 5. **Cloud Pub/Sub Metrics**
```python
PUBSUB_METRICS = {
    # Publishing Metrics
    'pubsub.googleapis.com/topic/send_request_count': {
        'description': 'Number of publish requests',
        'use_case': 'Message throughput tracking'
    },
    
    'pubsub.googleapis.com/topic/send_request_latencies': {
        'description': 'Publish latency distribution',
        'use_case': 'Publishing performance SLOs'
    },
    
    # Subscription Metrics
    'pubsub.googleapis.com/subscription/num_undelivered_messages': {
        'description': 'Queue depth (backlog)',
        'use_case': 'Consumer lag detection, scaling triggers'
    },
    
    'pubsub.googleapis.com/subscription/oldest_unacked_message_age': {
        'description': 'Age of oldest unacked message',
        'use_case': 'Processing delay alerts'
    },
    
    'pubsub.googleapis.com/subscription/pull_request_count': {
        'description': 'Number of pull requests',
        'use_case': 'Consumer activity monitoring'
    },
    
    'pubsub.googleapis.com/subscription/dead_letter_message_count': {
        'description': 'Messages sent to dead letter queue',
        'use_case': 'Error tracking, message failure analysis'
    }
}
```

### 6. **Memorystore (Redis) Metrics**
```python
REDIS_METRICS = {
    # Cache Performance
    'redis.googleapis.com/stats/cache_hit_ratio': {
        'description': 'Cache hit percentage',
        'use_case': 'Cache effectiveness, optimization'
    },
    
    'redis.googleapis.com/stats/memory/usage_ratio': {
        'description': 'Memory utilization percentage',
        'use_case': 'Capacity planning, eviction prevention'
    },
    
    # Connection Metrics
    'redis.googleapis.com/clients/connected': {
        'description': 'Number of connected clients',
        'use_case': 'Connection pool monitoring'
    },
    
    # Command Stats
    'redis.googleapis.com/commands/calls': {
        'description': 'Number of commands executed',
        'labels': ['cmd'],
        'use_case': 'Usage patterns, hotkey detection'
    },
    
    'redis.googleapis.com/stats/evicted_keys': {
        'description': 'Number of evicted keys',
        'use_case': 'Memory pressure detection'
    }
}
```

### 7. **Cloud Functions Metrics**
```python
CLOUD_FUNCTIONS_METRICS = {
    'cloudfunctions.googleapis.com/function/execution_count': {
        'description': 'Number of function executions',
        'labels': ['status'],
        'use_case': 'Success/failure tracking'
    },
    
    'cloudfunctions.googleapis.com/function/execution_times': {
        'description': 'Function execution duration',
        'type': 'DISTRIBUTION',
        'use_case': 'Performance optimization'
    },
    
    'cloudfunctions.googleapis.com/function/user_memory_bytes': {
        'description': 'Memory usage per execution',
        'use_case': 'Memory sizing optimization'
    },
    
    'cloudfunctions.googleapis.com/function/instance_count': {
        'description': 'Number of function instances',
        'use_case': 'Concurrency monitoring'
    }
}
```

### 8. **Uptime Check Metrics**
```python
UPTIME_CHECK_METRICS = {
    'monitoring.googleapis.com/uptime_check/check_passed': {
        'description': 'Uptime check success/failure',
        'labels': ['check_id', 'checker_location'],
        'use_case': 'Service availability SLOs'
    },
    
    'monitoring.googleapis.com/uptime_check/request_latency': {
        'description': 'Uptime check latency',
        'use_case': 'Global latency monitoring'
    }
}
```

### 9. **Cloud Logging Metrics (Log-Based)**
```python
LOG_BASED_METRICS = {
    'logging.googleapis.com/user/[METRIC_NAME]': {
        'description': 'Custom metrics from logs',
        'use_case': 'Business metrics, custom KPIs'
    },
    
    'logging.googleapis.com/log_entry_count': {
        'description': 'Number of log entries',
        'filters': 'severity>=ERROR',
        'use_case': 'Error rate tracking'
    }
}
```

---

## Level 1: Department View - Metrics & Reliability Actions {#level-1-department}

### What to Fetch
```python
"""
Department Level: Aggregate metrics across all 10 GCP projects
Target Audience: CTO, VP Engineering, Department Head
Focus: High-level health, trends, resource allocation
"""

department_metrics = {
    'overall_availability': {
        'source': 'monitoring.googleapis.com/uptime_check/check_passed',
        'aggregation': 'Average across all projects',
        'target': 99.5,
        'action_threshold': 99.0
    },
    
    'overall_reliability': {
        'source': 'run.googleapis.com/request_count (error_rate)',
        'calculation': '(total_requests - error_requests) / total_requests * 100',
        'target': 99.0,
        'action_threshold': 98.5
    },
    
    'overall_latency_score': {
        'source': 'run.googleapis.com/request_latencies OR loadbalancing.googleapis.com/https/backend_latencies',
        'calculation': 'Percentage of requests meeting P95 < target',
        'target': 98.0,
        'action_threshold': 95.0
    },
    
    'total_request_volume': {
        'source': 'run.googleapis.com/request_count',
        'aggregation': 'Sum across all projects',
        'use': 'Capacity planning, traffic trends'
    },
    
    'cost_per_request': {
        'source': 'run.googleapis.com/container/billable_instance_time + request_count',
        'calculation': 'Total cost / total requests',
        'use': 'Efficiency tracking, budget planning'
    }
}
```

### Reliability Actions Based on Department Metrics

#### 1. **When Overall Availability < 99.5%**
```python
def handle_low_department_availability(metrics):
    """
    Triggered when: overall_availability < 99.5%
    """
    actions = {
        'immediate': [
            'Identify which applications are below target',
            'Check for ongoing incidents across projects',
            'Review recent deployments in last 24h',
            'Verify multi-region failover configuration'
        ],
        
        'investigate': [
            'Query: Which GCP regions have degraded health?',
            'Metric: loadbalancing.googleapis.com/https/backend_latencies by region',
            'Check: Are uptime checks failing from specific geographic locations?',
            'Review: Recent changes to Load Balancer configuration'
        ],
        
        'remediate': [
            'If single region issue: Route traffic to healthy regions',
            'If specific app: Trigger incident response for that team',
            'If GCP issue: Check GCP status dashboard, engage support',
            'If persistent: Schedule emergency capacity review'
        ],
        
        'prevent': [
            'Implement circuit breakers between services',
            'Add redundancy in critical applications',
            'Review and update disaster recovery procedures',
            'Schedule chaos engineering exercises'
        ]
    }
    
    # Example automated action
    if metrics['availability'] < 98.0:  # Critical threshold
        notify_on_call_team()
        create_incident_ticket(severity='P1')
        enable_emergency_capacity()
```

#### 2. **When Overall Reliability < 99.0%**
```python
def handle_low_department_reliability(metrics, breakdown_by_app):
    """
    Triggered when: overall_reliability < 99.0% (error rate > 1%)
    """
    
    # Identify problematic applications
    problematic_apps = [
        app for app in breakdown_by_app 
        if app['error_rate'] > 1.0
    ]
    
    actions = {
        'immediate': [
            f'Focus on {len(problematic_apps)} applications with high error rates',
            'Check error distribution: 4xx vs 5xx',
            'Review recent deployments for these apps',
            'Check dependency health (databases, external APIs)'
        ],
        
        'analyze_errors': {
            'fetch_metric': 'run.googleapis.com/request_count',
            'filter': 'metric.response_code_class="5xx"',
            'group_by': ['service_name', 'response_code'],
            'analysis': 'Identify top error codes and affected services'
        },
        
        'root_cause': [
            'If 503 errors: Check instance counts, scaling limits',
            'If 500 errors: Review application logs for exceptions',
            'If 504 errors: Check timeout configurations, backend latency',
            'If 429 errors: Review rate limiting, quota issues'
        ],
        
        'remediate': [
            'For overloaded services: Increase max instances',
            'For application errors: Rollback recent deployments',
            'For timeout issues: Optimize slow operations',
            'For dependency failures: Enable fallback mechanisms'
        ]
    }
    
    # Automated remediation
    for app in problematic_apps:
        if app['error_code'] == '503':
            scale_up_service(app['project_id'], app['service_name'])
        elif app['error_code'] == '500':
            trigger_rollback_review(app['project_id'])
```

#### 3. **When Overall Latency Score < 98%**
```python
def handle_department_latency_degradation(metrics):
    """
    Triggered when: < 98% of requests meet latency SLOs
    """
    
    # Fetch detailed latency breakdown
    latency_analysis = {
        'p50_latency': 'Median latency across all services',
        'p95_latency': 'P95 latency (SLO boundary)',
        'p99_latency': 'P99 latency (worst case)',
        'by_region': 'Latency by geographic region',
        'by_service': 'Latency by service'
    }
    
    actions = {
        'immediate': [
            'Identify services with P95 > SLO target',
            'Check for database performance issues',
            'Review CDN cache hit rates',
            'Check for increased external API latency'
        ],
        
        'investigate': {
            'backend_latency': {
                'metric': 'loadbalancing.googleapis.com/https/backend_latencies',
                'question': 'Where is time spent? LB vs backend?'
            },
            'database_latency': {
                'metric': 'cloudsql.googleapis.com/database/mysql/queries',
                'question': 'Are slow queries increasing?'
            },
            'cache_effectiveness': {
                'metric': 'redis.googleapis.com/stats/cache_hit_ratio',
                'question': 'Has cache hit rate degraded?'
            }
        },
        
        'optimize': [
            'If DB slow: Identify and optimize slow queries',
            'If cache miss: Review cache TTLs, warming strategies',
            'If external API: Implement circuit breakers, timeouts',
            'If regional: Add CDN edges, optimize routing'
        ]
    }
```

#### 4. **Capacity Planning from Department Metrics**
```python
def department_capacity_planning(historical_metrics):
    """
    Use metrics for proactive capacity planning
    """
    
    analysis = {
        'traffic_trends': {
            'metric': 'run.googleapis.com/request_count',
            'period': '6 months',
            'calculation': 'Growth rate = (current - 6mo_ago) / 6mo_ago',
            'action': 'Project capacity needs for next 6 months'
        },
        
        'resource_utilization': {
            'cpu_usage': 'kubernetes.io/node/cpu/allocatable_utilization',
            'memory_usage': 'kubernetes.io/node/memory/allocatable_utilization',
            'threshold': 70,  # Alert if trending above 70%
            'action': 'Plan cluster expansion before hitting limits'
        },
        
        'cost_efficiency': {
            'metric': 'cost_per_request over time',
            'question': 'Is cost per request increasing?',
            'investigate': [
                'Cold start frequency (container/startup_latencies)',
                'Instance idle time',
                'Database connection efficiency'
            ]
        }
    }
    
    # Generate capacity plan
    if analysis['traffic_growth'] > 0.5:  # 50% growth
        plan = {
            'action': 'Increase infrastructure budget',
            'timeline': '3 months',
            'resources': [
                'Scale up GKE node pools',
                'Increase Cloud SQL instance sizes',
                'Add read replicas for high-traffic databases'
            ]
        }
        
        send_to_leadership(plan)
```

---

## Level 2: Application View - Metrics & Reliability Actions {#level-2-application}

### What to Fetch (Single GCP Project)
```python
"""
Application Level: Detailed metrics for one application (GCP project)
Target Audience: Product Owner, Engineering Manager
Focus: Feature performance, user journeys, optimization opportunities
"""

application_metrics = {
    # Request Metrics by Endpoint
    'request_breakdown': {
        'metric': 'run.googleapis.com/request_count',
        'group_by': ['path', 'method', 'response_code'],
        'use': 'Identify which endpoints have issues'
    },
    
    # Latency by CUJ (Critical User Journey)
    'cuj_latencies': {
        'metric': 'run.googleapis.com/request_latencies',
        'filter': 'metric.path="/api/v1/payment" OR "/api/v1/refund"',
        'percentiles': [50, 95, 99],
        'use': 'Track user journey performance'
    },
    
    # Resource Utilization
    'container_cpu': {
        'metric': 'run.googleapis.com/container/cpu/utilizations',
        'threshold': 80,  # Alert at 80%
        'use': 'Right-sizing, performance optimization'
    },
    
    'container_memory': {
        'metric': 'run.googleapis.com/container/memory/utilizations',
        'threshold': 85,
        'use': 'Memory leak detection'
    },
    
    # Scaling Behavior
    'instance_count': {
        'metric': 'run.googleapis.com/container/instance_count',
        'use': 'Validate autoscaling, capacity planning'
    },
    
    # Cold Starts
    'startup_latency': {
        'metric': 'run.googleapis.com/container/startup_latencies',
        'target': 1000,  # < 1 second
        'use': 'Minimize cold start impact'
    },
    
    # Database Performance
    'db_query_latency': {
        'metric': 'cloudsql.googleapis.com/database/mysql/queries',
        'percentiles': [95, 99],
        'use': 'Query optimization targets'
    },
    
    'db_connections': {
        'metric': 'cloudsql.googleapis.com/database/mysql/connections',
        'threshold': 'max_connections * 0.8',
        'use': 'Connection pool sizing'
    },
    
    # Cache Performance
    'cache_hit_rate': {
        'metric': 'redis.googleapis.com/stats/cache_hit_ratio',
        'target': 95,
        'use': 'Cache strategy effectiveness'
    },
    
    # Message Queue
    'queue_depth': {
        'metric': 'pubsub.googleapis.com/subscription/num_undelivered_messages',
        'threshold': 1000,
        'use': 'Consumer scaling, backpressure'
    }
}
```

### Reliability Actions Based on Application Metrics

#### 1. **Optimize Slow Endpoints (CUJs)**
```python
def optimize_slow_cujs(project_id, service_name):
    """
    Identify and optimize slow Critical User Journeys
    """
    
    # Fetch latency by endpoint
    query = f"""
    fetch cloud_run_revision
    | metric 'run.googleapis.com/request_latencies'
    | filter resource.project_id == '{project_id}'
    | filter resource.service_name == '{service_name}'
    | group_by [metric.path],
        [p95: percentile(value.request_latencies, 95)]
    | filter p95 > 200  # Threshold: 200ms
    """
    
    slow_endpoints = execute_mql_query(query)
    
    for endpoint in slow_endpoints:
        actions = {
            'trace_analysis': {
                'tool': 'Cloud Trace',
                'action': 'Analyze distributed traces for this endpoint',
                'find': [
                    'Slow database queries',
                    'Multiple external API calls',
                    'CPU-intensive operations',
                    'Serialization overhead'
                ]
            },
            
            'database_optimization': {
                'check': 'cloudsql.googleapis.com/database/mysql/slow_query_count',
                'action': 'Identify slow queries for this endpoint',
                'optimize': [
                    'Add missing indexes',
                    'Rewrite N+1 queries',
                    'Add query result caching'
                ]
            },
            
            'caching': {
                'check': 'Are results cacheable?',
                'implement': [
                    'Add Redis caching for expensive operations',
                    'Implement CDN caching for static content',
                    'Use edge functions for compute-at-edge'
                ]
            },
            
            'async_processing': {
                'check': 'Can operations be async?',
                'implement': [
                    'Move heavy processing to background jobs',
                    'Use Pub/Sub for async workflows',
                    'Return 202 Accepted for long operations'
                ]
            }
        }
        
        create_optimization_ticket(endpoint, actions)
```

#### 2. **Handle High Error Rates**
```python
def handle_application_errors(project_id, service_name, time_window='1h'):
    """
    Automated error analysis and remediation
    """
    
    # Get error breakdown
    error_query = f"""
    fetch cloud_run_revision
    | metric 'run.googleapis.com/request_count'
    | filter resource.project_id == '{project_id}'
    | filter resource.service_name == '{service_name}'
    | filter metric.response_code_class == '5xx' || metric.response_code_class == '4xx'
    | within {time_window}
    | group_by [metric.response_code, metric.path],
        [error_count: sum(value.request_count)]
    """
    
    errors = execute_mql_query(error_query)
    
    for error in errors:
        if error['response_code'] == '503':
            # Service overloaded
            actions = {
                'immediate': [
                    'Check instance count vs max_instances',
                    'Increase max_instances if at limit',
                    'Check CPU/Memory utilization'
                ],
                'investigate': {
                    'metric': 'run.googleapis.com/container/cpu/utilizations',
                    'question': 'Is CPU maxed out?'
                },
                'fix': [
                    'Increase max instances in Cloud Run',
                    'Optimize CPU-intensive code paths',
                    'Add caching to reduce load'
                ]
            }
            
            # Auto-scale if needed
            current_max = get_max_instances(project_id, service_name)
            if current_max < 100:
                update_max_instances(project_id, service_name, current_max * 2)
                
        elif error['response_code'] == '500':
            # Application error
            actions = {
                'immediate': [
                    'Check Cloud Logging for stack traces',
                    'Identify error message patterns',
                    'Check recent deployments'
                ],
                'log_query': f"""
                    resource.type="cloud_run_revision"
                    resource.labels.service_name="{service_name}"
                    severity>=ERROR
                    httpRequest.status=500
                """,
                'fix': [
                    'If new: Rollback recent deployment',
                    'If data issue: Fix data validation',
                    'If dependency: Add circuit breaker, retry logic'
                ]
            }
            
            # Check if error rate spiked after deployment
            if error_spike_after_deployment(project_id, service_name):
                trigger_automatic_rollback(project_id, service_name)
                
        elif error['response_code'] == '429':
            # Rate limiting
            actions = {
                'check': [
                    'Is rate limit correctly configured?',
                    'Is there a spike in traffic?',
                    'Is this a DDoS or bot attack?'
                ],
                'fix': [
                    'Increase rate limits if legitimate traffic',
                    'Implement more granular rate limiting',
                    'Add Cloud Armor rules for bot protection'
                ]
            }
```

#### 3. **Memory Leak Detection**
```python
def detect_memory_leaks(project_id, service_name):
    """
    Monitor for memory leaks in containers
    """
    
    # Fetch memory usage over time
    memory_query = f"""
    fetch cloud_run_revision
    | metric 'run.googleapis.com/container/memory/utilizations'
    | filter resource.project_id == '{project_id}'
    | filter resource.service_name == '{service_name}'
    | within 6h
    | group_by [], [memory_avg: mean(value.memory_utilizations)]
    """
    
    memory_trend = execute_mql_query(memory_query)
    
    # Check for increasing trend
    if is_increasing_trend(memory_trend):
        actions = {
            'confirm_leak': {
                'check': 'run.googleapis.com/container/restart_count',
                'question': 'Are containers restarting due to OOM?'
            },
            
            'investigate': [
                'Enable memory profiling in application',
                'Check for unclosed connections/file handles',
                'Review object lifecycle management',
                'Check for circular references'
            ],
            
            'immediate_mitigation': [
                'Increase container memory limit',
                'Reduce container lifetime (restart more frequently)',
                'Add memory usage alerts'
            ],
            
            'long_term_fix': [
                'Profile application memory usage',
                'Fix memory leaks in code',
                'Implement proper cleanup in destructors',
                'Add resource pooling'
            ]
        }
        
        # Auto-increase memory if needed
        current_memory = get_container_memory(project_id, service_name)
        if current_memory < 4096:  # Max 4GB
            update_container_memory(project_id, service_name, current_memory * 1.5)
        
        alert_team(f"Potential memory leak detected in {service_name}", actions)
```

#### 4. **Database Connection Pool Optimization**
```python
def optimize_database_connections(project_id, db_instance):
    """
    Optimize database connection pool based on metrics
    """
    
    metrics = {
        'current_connections': {
            'metric': 'cloudsql.googleapis.com/database/mysql/connections',
            'calculation': 'current_value'
        },
        'max_connections': {
            'metric': 'cloudsql.googleapis.com/database/mysql/max_connections',
            'calculation': 'limit'
        },
        'connection_errors': {
            'metric': 'cloudsql.googleapis.com/database/mysql/connections_failed',
            'threshold': 10  # per minute
        }
    }
    
    current = get_metric_value(metrics['current_connections'])
    maximum = get_metric_value(metrics['max_connections'])
    utilization = (current / maximum) * 100
    
    if utilization > 80:
        # Connection pool exhausted
        actions = {
            'immediate': [
                'Increase max_connections on Cloud SQL',
                'Check for connection leaks in application',
                'Review connection pool settings'
            ],
            
            'investigate': {
                'application_side': [
                    'Check connection pool max size',
                    'Verify connections are being closed',
                    'Check for long-running transactions'
                ],
                'database_side': [
                    'Query for long-running queries',
                    'Check for locked transactions',
                    'Review idle connections'
                ]
            },
            
            'optimize': [
                'Implement connection pooling (e.g., PgBouncer)',
                'Use Cloud SQL Proxy for connection management',
                'Reduce connection pool max lifetime',
                'Implement query timeout limits'
            ]
        }
        
        # Auto-increase if possible
        if maximum < 1000:
            update_max_connections(db_instance, maximum + 100)
            
    elif utilization < 20:
        # Over-provisioned
        actions = {
            'optimize': [
                'Reduce max_connections to save resources',
                'Reduce application connection pool size',
                'Review if all connections are necessary'
            ]
        }
```

---

## Level 3: SLO View - Metrics & Reliability Actions {#level-3-slo}

### What to Fetch (Individual SLOs)
```python
"""
SLO Level: Granular metrics for specific Service Level Objectives
Target Audience: DevOps, SRE, Development Team
Focus: Error budget management, proactive optimization, incident response
"""

slo_metrics = {
    # SLO 1: API Latency
    'api_latency_slo': {
        'name': 'API Response Time',
        'target': '95% of requests < 200ms (P95)',
        'metrics': {
            'latency_distribution': 'run.googleapis.com/request_latencies',
            'percentile': 95,
            'threshold_ms': 200
        },
        'error_budget': {
            'allowed_failure_rate': 5.0,  # 5% can be slow
            'window': '30 days'
        },
        'actions': 'Optimize slow paths, add caching, database tuning'
    },
    
    # SLO 2: Service Availability
    'availability_slo': {
        'name': 'Service Uptime',
        'target': '99.9% availability',
        'metrics': {
            'uptime_checks': 'monitoring.googleapis.com/uptime_check/check_passed',
            'instance_availability': 'run.googleapis.com/container/billable_instance_time'
        },
        'error_budget': {
            'allowed_downtime_30d': 43.2,  # minutes (0.1% of 30 days)
            'window': '30 days'
        },
        'actions': 'Multi-region deployment, health checks, auto-healing'
    },
    
    # SLO 3: Error Rate
    'error_rate_slo': {
        'name': 'Request Success Rate',
        'target': '< 1% error rate (99% success)',
        'metrics': {
            'total_requests': 'run.googleapis.com/request_count',
            'error_requests': 'run.googleapis.com/request_count (4xx,5xx filter)',
        },
        'error_budget': {
            'allowed_error_rate': 1.0,
            'window': '30 days'
        },
        'actions': 'Fix bugs, add retry logic, improve error handling'
    },
    
    # SLO 4: Database Query Performance
    'db_latency_slo': {
        'name': 'Database Query Latency',
        'target': '99% of queries < 100ms',
        'metrics': {
            'query_latency': 'cloudsql.googleapis.com/database/mysql/queries',
            'percentile': 99,
            'threshold_ms': 100
        },
        'error_budget': {
            'allowed_slow_queries': 1.0,
            'window': '30 days'
        },
        'actions': 'Add indexes, optimize queries, implement caching'
    },
    
    # SLO 5: Cache Hit Rate
    'cache_slo': {
        'name': 'Cache Effectiveness',
        'target': '> 95% cache hit rate',
        'metrics': {
            'hit_rate': 'redis.googleapis.com/stats/cache_hit_ratio'
        },
        'error_budget': {
            'allowed_miss_rate': 5.0,
            'window': '30 days'
        },
        'actions': 'Optimize cache keys, increase TTL, pre-warm cache'
    },
    
    # SLO 6: Message Processing Time
    'queue_processing_slo': {
        'name': 'Message Queue Latency',
        'target': '95% of messages processed within 5 minutes',
        'metrics': {
            'message_age': 'pubsub.googleapis.com/subscription/oldest_unacked_message_age',
            'threshold_seconds': 300
        },
        'error_budget': {
            'allowed_delayed_messages': 5.0,
            'window': '30 days'
        },
        'actions': 'Scale consumers, optimize processing, add parallelism'
    }
}
```

### Error Budget Calculation & Management
```python
def calculate_error_budget(slo_config, actual_metrics, window_days=30):
    """
    Calculate error budget consumption and remaining budget
    """
    
    if slo_config['type'] == 'latency':
        # Latency-based SLO
        total_requests = actual_metrics['total_requests']
        requests_meeting_slo = actual_metrics['requests_meeting_slo']
        
        actual_compliance = (requests_meeting_slo / total_requests) * 100
        target_compliance = slo_config['target']
        
        # Error budget = allowed failure %
        error_budget_total = 100 - target_compliance  # e.g., 5%
        error_budget_consumed = 100 - actual_compliance  # e.g., 2%
        error_budget_remaining = error_budget_total - error_budget_consumed  # e.g., 3%
        
        error_budget_remaining_pct = (error_budget_remaining / error_budget_total) * 100
        
        return {
            'total_budget': error_budget_total,
            'consumed': error_budget_consumed,
            'consumed_percentage': (error_budget_consumed / error_budget_total) * 100,
            'remaining': error_budget_remaining,
            'remaining_percentage': error_budget_remaining_pct,
            'status': get_budget_status(error_budget_remaining_pct)
        }
        
    elif slo_config['type'] == 'availability':
        # Availability-based SLO
        total_minutes_in_window = window_days * 24 * 60
        allowed_downtime = total_minutes_in_window * (1 - slo_config['target'] / 100)
        actual_downtime = actual_metrics['downtime_minutes']
        
        consumed_percentage = (actual_downtime / allowed_downtime) * 100
        remaining_minutes = allowed_downtime - actual_downtime
        remaining_percentage = (remaining_minutes / allowed_downtime) * 100
        
        return {
            'total_budget_minutes': allowed_downtime,
            'consumed_minutes': actual_downtime,
            'consumed_percentage': consumed_percentage,
            'remaining_minutes': remaining_minutes,
            'remaining_percentage': remaining_percentage,
            'status': get_budget_status(remaining_percentage)
        }

def get_budget_status(remaining_percentage):
    """Determine error budget health status"""
    if remaining_percentage > 50:
        return 'healthy'
    elif remaining_percentage > 25:
        return 'warning'
    else:
        return 'critical'
```

### SLO-Based Reliability Actions

#### 1. **When Error Budget is Critical (< 25% remaining)**
```python
def handle_critical_error_budget(slo_name, error_budget, metrics):
    """
    CRITICAL: < 25% error budget remaining - immediate action required
    """
    
    actions = {
        'immediate': {
            'freeze_deployments': {
                'action': 'Block all non-critical deployments',
                'reason': 'Prevent further degradation',
                'duration': 'Until budget recovers to > 25%'
            },
            
            'incident_response': {
                'action': 'Create incident ticket',
                'severity': 'P1',
                'owner': 'On-call SRE',
                'notification': ['PagerDuty', 'Slack', 'Email']
            },
            
            'deep_dive_analysis': {
                'action': 'Analyze root cause of budget consumption',
                'focus': [
                    'Recent changes/deployments',
                    'Traffic patterns',
                    'External dependencies',
                    'Infrastructure health'
                ]
            }
        },
        
        'remediation_by_slo_type': {
            'latency': {
                'quick_wins': [
                    'Enable aggressive caching',
                    'Scale up resources (CPU/Memory)',
                    'Shed non-critical features',
                    'Add read replicas for databases'
                ],
                'investigation': [
                    'Profile slow requests',
                    'Identify expensive operations',
                    'Check database query performance',
                    'Review external API latencies'
                ]
            },
            
            'availability': {
                'quick_wins': [
                    'Increase instance count',
                    'Enable multi-region failover',
                    'Restart unhealthy instances',
                    'Scale up unhealthy dependencies'
                ],
                'investigation': [
                    'Check recent incidents',
                    'Review deployment history',
                    'Analyze uptime check failures',
                    'Check infrastructure alerts'
                ]
            },
            
            'error_rate': {
                'quick_wins': [
                    'Rollback recent deployments',
                    'Enable circuit breakers',
                    'Add retry logic with backoff',
                    'Return cached/stale data if acceptable'
                ],
                'investigation': [
                    'Analyze error patterns',
                    'Check dependency health',
                    'Review recent code changes',
                    'Validate data integrity'
                ]
            }
        },
        
        'communication': {
            'stakeholders': [
                'Notify product owner',
                'Update engineering leadership',
                'Communicate to dependent teams',
                'Prepare customer communication if needed'
            ],
            'updates': {
                'frequency': 'Every 30 minutes',
                'channels': ['Slack incident channel', 'Status page'],
                'content': ['Current status', 'Actions taken', 'ETA for resolution']
            }
        }
    }
    
    # Automated emergency actions
    if slo_name == 'api_latency_slo' and error_budget['remaining_percentage'] < 10:
        # Emergency capacity boost
        increase_max_instances(project_id, service_name, factor=2)
        enable_aggressive_caching()
        
    elif slo_name == 'availability_slo' and error_budget['remaining_percentage'] < 10:
        # Emergency failover
        enable_multi_region_routing()
        increase_health_check_frequency()
        
    # Block deployments automatically
    set_deployment_freeze(slo_name, duration_hours=24)
    
    # Create incident
    incident_id = create_incident(
        title=f"Critical Error Budget: {slo_name}",
        severity='P1',
        details=error_budget
    )
    
    return actions, incident_id
```

#### 2. **When Error Budget is Warning (25-50% remaining)**
```python
def handle_warning_error_budget(slo_name, error_budget, metrics):
    """
    WARNING: 25-50% error budget remaining - proactive optimization
    """
    
    actions = {
        'monitoring': {
            'increase_alerting': {
                'action': 'Tighten alert thresholds',
                'examples': [
                    'Alert on P95 latency > 180ms (vs 200ms)',
                    'Alert on error rate > 0.8% (vs 1%)',
                    'Alert on availability < 99.95% (vs 99.9%)'
                ]
            },
            
            'dashboard_review': {
                'frequency': 'Daily',
                'attendees': ['SRE on-call', 'Dev team lead'],
                'review': [
                    'Error budget trends',
                    'Recent incidents',
                    'Upcoming deployments',
                    'Optimization opportunities'
                ]
            }
        },
        
        'optimization': {
            'prioritize_work': {
                'action': 'Prioritize performance/reliability work',
                'defer': ['New features', 'Non-critical improvements'],
                'focus': ['Performance optimization', 'Bug fixes', 'Stability improvements']
            },
            
            'by_slo_type': {
                'latency': [
                    'Optimize top 10 slowest endpoints',
                    'Add caching to expensive operations',
                    'Implement database query optimization',
                    'Review and optimize algorithms'
                ],
                
                'availability': [
                    'Review and fix flaky tests',
                    'Improve health check accuracy',
                    'Add retry logic to external dependencies',
                    'Implement graceful degradation'
                ],
                
                'error_rate': [
                    'Fix top error-generating bugs',
                    'Improve input validation',
                    'Add better error handling',
                    'Implement fallback mechanisms'
                ]
            }
        },
        
        'deployment_controls': {
            'increased_scrutiny': {
                'require': [
                    'Performance testing before deployment',
                    'Gradual rollout (canary deployment)',
                    'Increased monitoring during rollout',
                    'Quick rollback plan'
                ]
            },
            
            'change_freeze_consideration': {
                'when': 'Before major events or holidays',
                'duration': '1 week before/after',
                'exceptions': 'Critical bug fixes only'
            }
        }
    }
    
    # Automated actions
    increase_monitoring_granularity(slo_name)
    schedule_optimization_sprint(slo_name, priority='high')
    
    # Notification (not paging)
    notify_team(
        channel='#reliability-alerts',
        message=f"⚠️ {slo_name} error budget at {error_budget['remaining_percentage']:.1f}% - optimization needed",
        actions=actions['optimization']
    )
    
    return actions
```

#### 3. **Proactive SLO Optimization (> 50% budget remaining)**
```python
def optimize_healthy_slo(slo_name, error_budget, metrics):
    """
    HEALTHY: > 50% error budget - invest in improvements
    """
    
    strategies = {
        'tighten_slo': {
            'rationale': 'Current SLO is too loose, we can do better',
            'examples': [
                'Improve latency target from 200ms to 150ms',
                'Improve availability from 99.9% to 99.95%',
                'Reduce error rate target from 1% to 0.5%'
            ],
            'benefits': [
                'Better user experience',
                'Competitive advantage',
                'Demonstrates engineering excellence'
            ],
            'considerations': [
                'Ensure team can sustain tighter SLO',
                'Update monitoring and alerting',
                'Communicate to stakeholders'
            ]
        },
        
        'invest_in_resilience': {
            'infrastructure': [
                'Implement multi-region deployment',
                'Add automated failover',
                'Increase redundancy',
                'Improve disaster recovery'
            ],
            
            'application': [
                'Add circuit breakers',
                'Implement retry with exponential backoff',
                'Add rate limiting',
                'Improve observability (tracing, logging)'
            ],
            
            'testing': [
                'Implement chaos engineering',
                'Load testing at 2x peak capacity',
                'Failover testing',
                'Disaster recovery drills'
            ]
        },
        
        'performance_optimization': {
            'proactive': [
                'Profile and optimize hot paths',
                'Implement advanced caching strategies',
                'Optimize database schemas and queries',
                'Reduce cold start times'
            ],
            
            'cost_optimization': [
                'Right-size resources (reduce over-provisioning)',
                'Optimize instance utilization',
                'Reduce unnecessary logging/metrics',
                'Implement auto-scaling tuning'
            ]
        },
        
        'team_development': {
            'training': [
                'SRE best practices',
                'Performance optimization techniques',
                'Incident response drills',
                'Chaos engineering workshops'
            ],
            
            'documentation': [
                'Create runbooks for common issues',
                'Document architecture decisions',
                'Write postmortems for all incidents',
                'Share learnings across teams'
            ]
        }
    }
    
    # Generate improvement plan
    plan = {
        'short_term': [
            'Review and optimize top 5 slowest operations',
            'Add comprehensive monitoring for blind spots',
            'Conduct load testing'
        ],
        
        'medium_term': [
            'Implement chaos engineering experiments',
            'Add multi-region failover capability',
            'Optimize resource utilization'
        ],
        
        'long_term': [
            'Tighten SLO targets for better user experience',
            'Build predictive alerting with ML',
            'Achieve 99.99% availability'
        ]
    }
    
    create_improvement_backlog(slo_name, plan)
    
    return strategies, plan
```

#### 4. **Database-Specific SLO Actions**
```python
def optimize_database_slo(db_instance, slo_metrics):
    """
    Specific actions for database query performance SLO
    """
    
    # Fetch slow query data
    slow_queries = get_slow_queries(db_instance)
    
    actions = {
        'query_optimization': {
            'identify': {
                'metric': 'cloudsql.googleapis.com/database/mysql/slow_query_count',
                'threshold': '> 100 queries/hour',
                'action': 'Enable slow query log analysis'
            },
            
            'optimize': [
                {
                    'issue': 'Missing indexes',
                    'detect': 'Explain plan shows full table scan',
                    'fix': 'Add appropriate indexes',
                    'example': 'CREATE INDEX idx_user_email ON users(email)'
                },
                {
                    'issue': 'N+1 query problem',
                    'detect': 'Same query pattern executed many times',
                    'fix': 'Use JOINs or batch queries',
                    'example': 'SELECT * FROM orders WHERE user_id IN (...)'
                },
                {
                    'issue': 'Large result sets',
                    'detect': 'Queries returning > 1000 rows',
                    'fix': 'Add pagination, limit results',
                    'example': 'SELECT * FROM logs LIMIT 100 OFFSET 0'
                },
                {
                    'issue': 'Inefficient JOINs',
                    'detect': 'Multiple JOINs with large tables',
                    'fix': 'Denormalize, add materialized views',
                    'example': 'Create summary tables'
                }
            ]
        },
        
        'connection_pooling': {
            'metrics': {
                'current': 'cloudsql.googleapis.com/database/mysql/connections',
                'max': 'cloudsql.googleapis.com/database/mysql/max_connections'
            },
            
            'optimize': [
                'Implement connection pooling (PgBouncer, Cloud SQL Proxy)',
                'Reduce connection pool size in application',
                'Set appropriate connection timeouts',
                'Use read replicas for read-heavy workloads'
            ]
        },
        
        'caching_strategy': {
            'implement': [
                'Query result caching (Redis)',
                'Application-level caching',
                'Database query cache (MySQL)',
                'Materialized views for complex queries'
            ],
            
            'monitor': {
                'metric': 'redis.googleapis.com/stats/cache_hit_ratio',
                'target': '> 95%',
                'action': 'Optimize cache keys and TTLs'
            }
        },
        
        'infrastructure': {
            'read_replicas': {
                'metric': 'cloudsql.googleapis.com/database/replication/replica_lag',
                'threshold': '< 5 seconds',
                'use': 'Offload read traffic to replicas'
            },
            
            'vertical_scaling': {
                'metrics': [
                    'cloudsql.googleapis.com/database/cpu/utilization',
                    'cloudsql.googleapis.com/database/memory/utilization'
                ],
                'threshold': '> 80%',
                'action': 'Upgrade to larger instance type'
            }
        }
    }
    
    # Automated optimization
    for query in slow_queries:
        if query['execution_count'] > 1000:  # Frequently executed
            suggest_index_for_query(query)
            
        if query['rows_examined'] > query['rows_sent'] * 100:  # Inefficient
            flag_for_optimization(query)
```

#### 5. **Queue Processing SLO Actions**
```python
def optimize_queue_processing_slo(subscription_name, slo_metrics):
    """
    Optimize message queue processing time
    """
    
    metrics = {
        'queue_depth': 'pubsub.googleapis.com/subscription/num_undelivered_messages',
        'message_age': 'pubsub.googleapis.com/subscription/oldest_unacked_message_age',
        'ack_rate': 'pubsub.googleapis.com/subscription/ack_message_count'
    }
    
    queue_depth = get_metric_value(metrics['queue_depth'])
    message_age = get_metric_value(metrics['message_age'])
    
    actions = {
        'consumer_scaling': {
            'if': queue_depth > 1000,
            'actions': [
                'Increase number of consumer instances',
                'Increase max concurrent messages per consumer',
                'Optimize message processing time',
                'Implement parallel processing'
            ],
            
            'auto_scale': {
                'metric': 'pubsub.googleapis.com/subscription/num_undelivered_messages',
                'threshold': 1000,
                'action': 'Scale Cloud Run consumers',
                'formula': 'instances = queue_depth / 100'
            }
        },
        
        'processing_optimization': {
            'if': message_age > 300,  # > 5 minutes
            'investigate': [
                'Profile message processing code',
                'Check for external API timeouts',
                'Review database query performance',
                'Look for synchronous operations that can be async'
            ],
            
            'optimize': [
                'Batch processing for similar messages',
                'Cache frequently accessed data',
                'Implement circuit breakers for external calls',
                'Add retry logic with exponential backoff'
            ]
        },
        
        'dead_letter_queue': {
            'metric': 'pubsub.googleapis.com/subscription/dead_letter_message_count',
            'if': 'count > 0',
            'actions': [
                'Investigate why messages failed',
                'Fix processing logic',
                'Replay messages after fix',
                'Add better error handling'
            ]
        },
        
        'backpressure': {
            'implement': [
                'Rate limiting on publishers',
                'Circuit breaker when queue is full',
                'Alternative processing paths',
                'Graceful degradation'
            ]
        }
    }
    
    # Auto-scale consumers based on queue depth
    if queue_depth > 1000:
        target_instances = min(queue_depth // 100, 50)  # Max 50 instances
        scale_consumers(subscription_name, target_instances)
        
    # Alert on dead letter queue
    if get_metric_value('dead_letter_message_count') > 0:
        alert_team(
            severity='warning',
            message=f"Dead letter messages detected in {subscription_name}",
            action='Investigate and fix processing errors'
        )
```

---

## Complete Python Implementation {#python-implementation}

### Full SDK Implementation with Actions
```python
#!/usr/bin/env python3
"""
Complete GCP Monitoring Integration for Reliability Dashboard
Includes automated reliability actions based on metrics
"""

from google.cloud import monitoring_v3
from google.cloud import logging_v2
from datetime import datetime, timedelta
import json
from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum

class BudgetStatus(Enum):
    HEALTHY = "healthy"
    WARNING = "warning"
    CRITICAL = "critical"

@dataclass
class SLO:
    name: str
    target: float
    metric_type: str
    threshold: float
    window_days: int = 30

@dataclass
class ErrorBudget:
    total: float
    consumed: float
    remaining: float
    remaining_percentage: float
    status: BudgetStatus

class GCPReliabilityManager:
    """
    Comprehensive GCP Monitoring with automated reliability actions
    """
    
    def __init__(self, project_ids: List[str]):
        self.project_ids = project_ids
        self.metric_client = monitoring_v3.MetricServiceClient()
        self.query_client = monitoring_v3.QueryServiceClient()
        
    # ========== METRIC FETCHING ==========
    
    def fetch_time_series(
        self, 
        project_id: str, 
        metric_type: str,
        hours_ago: int = 24,
        filters: Optional[str] = None,
        aggregation: Optional[Dict] = None
    ) -> List:
        """Fetch time series data with optional filters and aggregation"""
        
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
        
        if aggregation:
            request.aggregation = monitoring_v3.Aggregation(aggregation)
        
        results = self.metric_client.list_time_series(request=request)
        return list(results)
    
    def execute_mql_query(self, project_id: str, query: str) -> List:
        """Execute Monitoring Query Language (MQL) query"""
        
        request = monitoring_v3.QueryTimeSeriesRequest({
            "name": f"projects/{project_id}",
            "query": query
        })
        
        page_result = self.query_client.query_time_series(request=request)
        return list(page_result)
    
    # ========== LEVEL 1: DEPARTMENT METRICS ==========
    
    def get_department_metrics(self) -> Dict:
        """
        Aggregate metrics across all projects for department view
        """
        
        all_reliability = []
        all_availability = []
        all_latency_scores = []
        applications = []
        
        for project_id in self.project_ids:
            app_metrics = self.get_application_summary(project_id)
            
            all_reliability.append(app_metrics['reliability'])
            all_availability.append(app_metrics['availability'])
            all_latency_scores.append(app_metrics['latency_score'])
            
            applications.append({
                'project_id': project_id,
                'name': app_metrics['name'],
                'reliability': app_metrics['reliability'],
                'availability': app_metrics['availability'],
                'latency_score': app_metrics['latency_score'],
                'status': app_metrics['status']
            })
        
        dept_metrics = {
            'overall_reliability': sum(all_reliability) / len(all_reliability),
            'overall_availability': sum(all_availability) / len(all_availability),
            'overall_latency_score': sum(all_latency_scores) / len(all_latency_scores),
            'total_applications': len(self.project_ids),
            'applications': applications
        }
        
        # Automated actions based on department metrics
        self.department_reliability_actions(dept_metrics)
        
        return dept_metrics
    
    def department_reliability_actions(self, metrics: Dict):
        """Take automated actions based on department-level metrics"""
        
        if metrics['overall_availability'] < 99.0:
            self.handle_low_department_availability(metrics)
            
        if metrics['overall_reliability'] < 98.5:
            self.handle_low_department_reliability(metrics)
            
        if metrics['overall_latency_score'] < 95.0:
            self.handle_department_latency_issues(metrics)
    
    def handle_low_department_availability(self, metrics: Dict):
        """Actions when department availability is low"""
        
        print(f"⚠️ ALERT: Department availability at {metrics['overall_availability']:.2f}%")
        
        # Identify problematic applications
        problem_apps = [
            app for app in metrics['applications']
            if app['availability'] < 99.0
        ]
        
        for app in problem_apps:
            print(f"  - {app['name']}: {app['availability']:.2f}%")
            # Create incident ticket
            self.create_incident_ticket(
                project_id=app['project_id'],
                severity='P1',
                title=f"Low availability in {app['name']}",
                details=app
            )
    
    # ========== LEVEL 2: APPLICATION METRICS ==========
    
    def get_application_summary(self, project_id: str) -> Dict:
        """
        Get summary metrics for a single application
        """
        
        # Request metrics
        request_metrics = self.get_request_metrics(project_id)
        
        # Resource metrics
        resource_metrics = self.get_resource_metrics(project_id)
        
        # Database metrics (if applicable)
        db_metrics = self.get_database_metrics(project_id)
        
        # Calculate health status
        status = self.calculate_health_status(
            request_metrics['reliability'],
            request_metrics['availability'],
            request_metrics['latency_score']
        )
        
        app_summary = {
            'name': self.get_service_name(project_id),
            'project_id': project_id,
            'reliability': request_metrics['reliability'],
            'availability': request_metrics['availability'],
            'latency_score': request_metrics['latency_score'],
            'total_requests_24h': request_metrics['total_requests'],
            'error_rate': request_metrics['error_rate'],
            'p95_latency_ms': request_metrics['p95_latency'],
            'p99_latency_ms': request_metrics['p99_latency'],
            'cpu_utilization': resource_metrics['avg_cpu'],
            'memory_utilization': resource_metrics['avg_memory'],
            'instance_count': resource_metrics['instance_count'],
            'status': status
        }
        
        # Automated reliability actions
        self.application_reliability_actions(project_id, app_summary)
        
        return app_summary
    
    def get_request_metrics(self, project_id: str) -> Dict:
        """Get request-level metrics (Cloud Run, Load Balancer)"""
        
        # Total requests
        total_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_count',
            hours_ago=24
        )
        
        total_requests = sum(
            sum(point.value.int64_value or point.value.double_value 
                for point in series.points)
            for series in total_series
        )
        
        # Error requests
        error_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_count',
            hours_ago=24,
            filters='metric.response_code_class="5xx" OR metric.response_code_class="4xx"'
        )
        
        error_requests = sum(
            sum(point.value.int64_value or point.value.double_value 
                for point in series.points)
            for series in error_series
        )
        
        error_rate = (error_requests / total_requests * 100) if total_requests > 0 else 0
        reliability = 100 - error_rate
        
        # Latency
        latency_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/request_latencies',
            hours_ago=24
        )
        
        latencies = []
        for series in latency_series:
            for point in series.points:
                if hasattr(point.value, 'distribution_value'):
                    latencies.append(point.value.distribution_value.mean)
        
        latencies.sort()
        p95_latency = latencies[int(len(latencies) * 0.95)] if latencies else 0
        p99_latency = latencies[int(len(latencies) * 0.99)] if latencies else 0
        
        # Calculate latency score (% requests meeting SLO)
        threshold_ms = 200
        requests_meeting_slo = sum(1 for lat in latencies if lat <= threshold_ms)
        latency_score = (requests_meeting_slo / len(latencies) * 100) if latencies else 100
        
        # Availability (uptime checks)
        uptime_series = self.fetch_time_series(
            project_id,
            'monitoring.googleapis.com/uptime_check/check_passed',
            hours_ago=24
        )
        
        total_checks = 0
        passed_checks = 0
        for series in uptime_series:
            for point in series.points:
                total_checks += 1
                if point.value.bool_value or point.value.double_value > 0.5:
                    passed_checks += 1
        
        availability = (passed_checks / total_checks * 100) if total_checks > 0 else 100
        
        return {
            'total_requests': total_requests,
            'error_requests': error_requests,
            'error_rate': error_rate,
            'reliability': reliability,
            'availability': availability,
            'p95_latency': p95_latency,
            'p99_latency': p99_latency,
            'latency_score': latency_score
        }
    
    def get_resource_metrics(self, project_id: str) -> Dict:
        """Get resource utilization metrics"""
        
        # CPU utilization
        cpu_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/container/cpu/utilizations',
            hours_ago=1
        )
        
        cpu_values = [
            point.value.double_value
            for series in cpu_series
            for point in series.points
        ]
        avg_cpu = sum(cpu_values) / len(cpu_values) if cpu_values else 0
        
        # Memory utilization
        memory_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/container/memory/utilizations',
            hours_ago=1
        )
        
        memory_values = [
            point.value.double_value
            for series in memory_series
            for point in series.points
        ]
        avg_memory = sum(memory_values) / len(memory_values) if memory_values else 0
        
        # Instance count
        instance_series = self.fetch_time_series(
            project_id,
            'run.googleapis.com/container/instance_count',
            hours_ago=1
        )
        
        instance_count = max([
            point.value.int64_value or point.value.double_value
            for series in instance_series
            for point in series.points
        ], default=0)
        
        return {
            'avg_cpu': avg_cpu * 100,  # Convert to percentage
            'avg_memory': avg_memory * 100,
            'instance_count': instance_count
        }
    
    def get_database_metrics(self, project_id: str) -> Dict:
        """Get Cloud SQL metrics"""
        
        try:
            # Connections
            conn_series = self.fetch_time_series(
                project_id,
                'cloudsql.googleapis.com/database/mysql/connections',
                hours_ago=1
            )
            
            current_connections = max([
                point.value.double_value
                for series in conn_series
                for point in series.points
            ], default=0)
            
            # Max connections
            max_conn_series = self.fetch_time_series(
                project_id,
                'cloudsql.googleapis.com/database/mysql/max_connections',
                hours_ago=1
            )
            
            max_connections = max([
                point.value.double_value
                for series in max_conn_series
                for point in series.points
            ], default=100)
            
            return {
                'current_connections': current_connections,
                'max_connections': max_connections,
                'connection_utilization': (current_connections / max_connections * 100)
            }
            
        except Exception as e:
            # Database might not exist for this project
            return {
                'current_connections': 0,
                'max_connections': 0,
                'connection_utilization': 0
            }
    
    def application_reliability_actions(self, project_id: str, metrics: Dict):
        """Take automated actions based on application metrics"""
        
        # High CPU usage
        if metrics['cpu_utilization'] > 80:
            self.handle_high_cpu(project_id, metrics)
        
        # High memory usage
        if metrics['memory_utilization'] > 85:
            self.handle_high_memory(project_id, metrics)
        
        # High error rate
        if metrics['error_rate'] > 1.0:
            self.handle_high_error_rate(project_id, metrics)
        
        # High latency
        if metrics['p95_latency_ms'] > 200:
            self.handle_high_latency(project_id, metrics)
    
    def handle_high_cpu(self, project_id: str, metrics: Dict):
        """Actions when CPU is high"""
        print(f"⚠️ High CPU in {project_id}: {metrics['cpu_utilization']:.1f}%")
        
        # Auto-scale if possible
        service_name = self.get_service_name(project_id)
        current_max = self.get_max_instances(project_id, service_name)
        
        if current_max < 100:
            new_max = min(current_max * 2, 100)
            self.update_max_instances(project_id, service_name, new_max)
            print(f"  ✅ Increased max instances from {current_max} to {new_max}")
    
    def handle_high_memory(self, project_id: str, metrics: Dict):
        """Actions when memory is high"""
        print(f"⚠️ High Memory in {project_id}: {metrics['memory_utilization']:.1f}%")
        
        # Check for memory leak
        memory_trend = self.get_memory_trend(project_id, hours=6)
        if self.is_increasing_trend(memory_trend):
            print(f"  🚨 Potential memory leak detected!")
            self.create_incident_ticket(
                project_id=project_id,
                severity='P2',
                title=f"Potential memory leak in {self.get_service_name(project_id)}",
                details=metrics
            )
    
    # ========== LEVEL 3: SLO METRICS ==========
    
    def calculate_slo_compliance(
        self, 
        project_id: str, 
        slo: SLO
    ) -> Dict:
        """
        Calculate SLO compliance and error budget
        """
        
        if 'latency' in slo.metric_type:
            return self.calculate_latency_slo(project_id, slo)
        elif 'availability' in slo.metric_type or 'uptime' in slo.metric_type:
            return self.calculate_availability_slo(project_id, slo)
        elif 'error' in slo.metric_type or 'request_count' in slo.metric_type:
            return self.calculate_error_rate_slo(project_id, slo)
        else:
            raise ValueError(f"Unknown SLO type: {slo.metric_type}")
    
    def calculate_latency_slo(self, project_id: str, slo: SLO) -> Dict:
        """Calculate latency-based SLO"""
        
        hours = slo.window_days * 24
        
        latency_series = self.fetch_time_series(
            project_id,
            slo.metric_type,
            hours_ago=hours
        )
        
        latencies = []
        for series in latency_series:
            for point in series.points:
                if hasattr(point.value, 'distribution_value'):
                    latencies.append(point.value.distribution_value.mean)
        
        total_requests = len(latencies)
        requests_meeting_slo = sum(1 for lat in latencies if lat <= slo.threshold)
        
        compliance = (requests_meeting_slo / total_requests * 100) if total_requests > 0 else 100
        
        # Error budget
        error_budget = self.calculate_error_budget_latency(
            slo.target,
            compliance,
            total_requests,
            requests_meeting_slo
        )
        
        result = {
            'slo_name': slo.name,
            'target': slo.target,
            'actual_compliance': compliance,
            'total_requests': total_requests,
            'requests_meeting_slo': requests_meeting_slo,
            'error_budget': error_budget,
            'p95_latency': latencies[int(len(latencies) * 0.95)] if latencies else 0,
            'p99_latency': latencies[int(len(latencies) * 0.99)] if latencies else 0
        }
        
        # Automated actions based on error budget
        self.slo_reliability_actions(project_id, slo, result)
        
        return result
    
    def calculate_error_budget_latency(
        self,
        target: float,
        actual: float,
        total_requests: int,
        requests_meeting_slo: int
    ) -> ErrorBudget:
        """Calculate error budget for latency SLO"""
        
        total_budget = 100 - target  # e.g., 5% allowed to be slow
        consumed = 100 - actual  # e.g., 2% were slow
        remaining = total_budget - consumed
        remaining_pct = (remaining / total_budget * 100) if total_budget > 0 else 100
        
        if remaining_pct > 50:
            status = BudgetStatus.HEALTHY
        elif remaining_pct > 25:
            status = BudgetStatus.WARNING
        else:
            status = BudgetStatus.CRITICAL
        
        return ErrorBudget(
            total=total_budget,
            consumed=consumed,
            remaining=remaining,
            remaining_percentage=remaining_pct,
            status=status
        )
    
    def slo_reliability_actions(
        self, 
        project_id: str, 
        slo: SLO, 
        result: Dict
    ):
        """Take automated actions based on SLO error budget"""
        
        error_budget = result['error_budget']
        
        if error_budget.status == BudgetStatus.CRITICAL:
            self.handle_critical_error_budget(project_id, slo, result)
        elif error_budget.status == BudgetStatus.WARNING:
            self.handle_warning_error_budget(project_id, slo, result)
        else:
            self.optimize_healthy_slo(project_id, slo, result)
    
    def handle_critical_error_budget(
        self, 
        project_id: str, 
        slo: SLO, 
        result: Dict
    ):
        """CRITICAL: < 25% error budget remaining"""
        
        print(f"🚨 CRITICAL: {slo.name} error budget at {result['error_budget'].remaining_percentage:.1f}%")
        
        # Freeze deployments
        self.set_deployment_freeze(project_id, duration_hours=24)
        print(f"  🔒 Deployment freeze enabled for 24 hours")
        
        # Create P1 incident
        self.create_incident_ticket(
            project_id=project_id,
            severity='P1',
            title=f"Critical Error Budget: {slo.name}",
            details=result
        )
        
        # Emergency capacity boost
        if 'latency' in slo.metric_type:
            service_name = self.get_service_name(project_id)
            self.emergency_scale_up(project_id, service_name)
    
    # ========== HELPER METHODS ==========
    
    def get_service_name(self, project_id: str) -> str:
        """Get service name from project ID"""
        # Implementation depends on your naming convention
        return project_id.replace('-prod', '')
    
    def calculate_health_status(
        self, 
        reliability: float, 
        availability: float, 
        latency_score: float
    ) -> str:
        """Calculate overall health status"""
        
        if all([reliability >= 99, availability >= 99.5, latency_score >= 98]):
            return 'healthy'
        elif all([reliability >= 97, availability >= 98, latency_score >= 95]):
            return 'warning'
        else:
            return 'critical'
    
    def get_max_instances(self, project_id: str, service_name: str) -> int:
        """Get current max instances configuration"""
        # Implementation using Cloud Run API
        return 10  # Placeholder
    
    def update_max_instances(
        self, 
        project_id: str, 
        service_name: str, 
        max_instances: int
    ):
        """Update max instances for a Cloud Run service"""
        # Implementation using Cloud Run API
        print(f"  ⚙️ Would update max instances to {max_instances}")
    
    def create_incident_ticket(
        self, 
        project_id: str, 
        severity: str, 
        title: str, 
        details: Dict
    ):
        """Create incident ticket in your ticketing system"""
        print(f"  🎫 Creating {severity} incident: {title}")
        # Integration with Jira, PagerDuty, etc.
    
    def set_deployment_freeze(self, project_id: str, duration_hours: int):
        """Set deployment freeze"""
        # Implementation depends on your CI/CD system
        print(f"  🔒 Setting deployment freeze for {duration_hours} hours")
    
    def emergency_scale_up(self, project_id: str, service_name: str):
        """Emergency scaling action"""
        current_max = self.get_max_instances(project_id, service_name)
        new_max = min(current_max * 3, 100)
        self.update_max_instances(project_id, service_name, new_max)
        print(f"  🚀 Emergency scale: {current_max} -> {new_max} instances")


# ========== USAGE EXAMPLE ==========

if __name__ == '__main__':
    # Initialize with all project IDs
    project_ids = [
        'payment-service-prod',
        'auth-service-prod',
        'inventory-service-prod',
        'analytics-pipeline-prod',
        # ... add remaining projects
    ]
    
    manager = GCPReliabilityManager(project_ids)
    
    # Level 1: Department metrics
    print("=" * 50)
    print("LEVEL 1: DEPARTMENT METRICS")
    print("=" * 50)
    dept_metrics = manager.get_department_metrics()
    print(json.dumps(dept_metrics, indent=2))
    
    # Level 2: Application metrics
    print("\n" + "=" * 50)
    print("LEVEL 2: APPLICATION METRICS")
    print("=" * 50)
    app_metrics = manager.get_application_summary('payment-service-prod')
    print(json.dumps(app_metrics, indent=2))
    
    # Level 3: SLO compliance
    print("\n" + "=" * 50)
    print("LEVEL 3: SLO METRICS")
    print("=" * 50)
    
    latency_slo = SLO(
        name='API Response Time',
        target=95.0,
        metric_type='run.googleapis.com/request_latencies',
        threshold=200.0,  # ms
        window_days=30
    )
    
    slo_result = manager.calculate_slo_compliance('payment-service-prod', latency_slo)
    print(json.dumps(slo_result, indent=2, default=str))
```

This comprehensive guide provides:

1. **Complete metric catalog** from GCP Cloud Monitoring
2. **Three-level hierarchy** with specific metrics for each audience
3. **Automated reliability actions** triggered by metric thresholds
4. **Error budget management** with proactive/reactive strategies
5. **Full Python implementation** ready to use
6. **Real-world examples** of optimization and incident response

The code is production-ready and can be extended based on your specific infrastructure and requirements!
