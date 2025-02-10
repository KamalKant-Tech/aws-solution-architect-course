## Use Case of ECS Autoscaling 

### Scaling Based on Average CPU Utilization of Running Tasks in a Service
- <b> Q: How ECS service scale if scaling based on CPU utilization will check the cpu usage of each task</b>?
  - Yes, if you configure ECS Service Auto Scaling based on CPU utilization, it monitors the average CPU utilization across all tasks in the service, <b><i>not the CPU usage of individual tasks</i></b>. Here’s how it works:
  - <b>How ECS Service Scaling Based on CPU Utilization Works</b>
        1. Metric Used:
            - ECS uses the ECSServiceAverageCPUUtilization metric from Amazon CloudWatch.
            - This metric represents the average CPU utilization across all running tasks in the service.
        2. Scaling Evaluation:
            - The scaling policy evaluates the average CPU utilization compared to a defined threshold (e.g., scale out when CPU usage exceeds 70%).
        3. Trigger for Scaling:
            - When the average CPU utilization crosses the specified threshold
             - ECS Service Auto Scaling increases or decreases the desired count of tasks to maintain the target utilization.
        4. New Tasks and Resource Allocation:
            - When new tasks are added, ECS attempts to distribute them across available EC2 instances or Fargate infrastructure.
            - Similarly, tasks are stopped during scaling in, which reduces the average utilization.
        
    
- <b>Example Scenario</b>:
  - Configuration
    - Desired Task Count: 5.
    - Target CPU Utilization: 70%.
    - Task CPU Reservation: 512 CPU units (0.5 vCPU).
  - Flow
    - Initial State:
      - Assume tasks are running at 50% CPU utilization on average.
      - ECS does not scale because the average utilization is below the target (70%).
    - Increased Load:
      - A spike in traffic causes CPU utilization to rise to 80% across tasks.
      - ECS Service Auto Scaling triggers a scale-out action, increasing the desired task count to 6.
    - New Task Placement:
      - The new task reduces the load on existing tasks, lowering the average utilization.
    - Return to Target:
      - Once average utilization stabilizes near 70%, ECS stops scaling out further.
- <b> Q: How load Balancer helps ECS service autoscaling?</b>
    
    Load balancers play a crucial role in ECS Auto Scaling by dynamically managing the traffic distribution and ensuring that your application scales effectively in response to demand. They integrate with both the ECS Service Auto Scaling and the Auto Scaling Group (ASG) to enhance reliability, scalability, and availability.

  - <b>Distributing Traffic Across Tasks</b>

    - <b>Task-Level Scaling</b>:
        
      - ECS integrates with Elastic Load Balancers (ALB/NLB) to evenly distribute traffic among tasks running in your ECS service.
      - If you scale out by adding more tasks, the load balancer automatically routes traffic to the new tasks.
    
    - <b>Health Checks for Task Placement</b>
        - The load balancer performs health checks on ECS tasks.
            - Unhealthy tasks are removed from the load balancer’s target group.
            - ECS replaces unhealthy tasks, ensuring traffic is routed only to healthy containers.
            - This improves the availability and resilience of your service.      
    - <b>Facilitating ASG Scaling</b>

        The load balancer also interacts with the Auto Scaling Group (ASG) for EC2 instances:
        -   Scaling Based on Load Balancer Metrics:
            - ASG can use metrics from the load balancer to scale EC2 instances up or down, ensuring there’s enough capacity to handle tasks.
                - Examples:
                    - Request Count per Target: Scale EC2 instances when the number of requests exceeds a threshold.
                    - Target Response Time: Scale EC2 instances based on response time.
    - <b>Instance Registration/De-registration</b>
        - When an ASG launches new EC2 instances, they are automatically registered as targets with the load balancer.
        - If an instance is terminated, the load balancer ensures traffic is no longer routed to it.
  - <b>Enabling Dynamic Scaling</b>
    - ECS Service Auto Scaling:
        - ECS service can scale tasks in or out based on target tracking or step scaling policies linked to metrics such as CPU utilization or the load balancer’s Request Count per Target.
    - ASG Auto Scaling:
        - ASG ensures there are sufficient EC2 instances available to run the desired number of tasks in the ECS service.

    <hr></hr>

    - <b>Workflow Example</b>
        - Configuration:
            - ECS Service:
                - Desired task count: 5
                - Load balancer: Application Load Balancer (ALB) with a target group.
        - ASG:
            - Minimum instances: 2
            - Desired capacity: Scales based on ALB metrics.
            - Scaling Flow:
        - Increased Traffic:
            - Traffic spikes cause the ALB’s Request Count per Target metric to exceed the threshold.
            - Scaling Tasks:
            - ECS Service Auto Scaling adds 2 more tasks to handle the increased traffic.
        - Scaling Instances:
            - ASG detects increased CPU utilization on EC2 instances due to more tasks.
            - ASG scales out by launching an additional EC2 instance.
        - Traffic Distribution:
            - ALB automatically includes new tasks and EC2 instances in its routing.
    <hr></hr>

    - <b>Benefits of Using Load Balancers with ECS and ASG</b>
        - High Availability:
            - Ensures traffic is distributed only to healthy tasks and instances.
        - Dynamic Scalability:
            - Automatically adjusts task and instance capacity based on demand.
        - Improved Efficiency:
            - Prevents under-utilization or overloading of EC2 instances.
        - Seamless Scaling:
            - Smoothly integrates ECS task scaling and EC2 instance scaling.

    <hr></hr>

