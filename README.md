# terraform-module-ecs-service
Terraform module for creating an ECS service.

## Deploying a service with a Load Balancer
```hcl
module "task_with_alb" {
  source  = "lazzurs/ecs-service/aws"
  version = "0.4.0"
  ecs_cluster_id = "arn:aws:ecs:us-east-1:888888888888:cluster/ecs-0"
  image_name = "nginx:latest"
  service_name = "my-web-server"
  tld = "austincloud.guru"
  service_memory = 2048
  mount_points = [
    {
      sourceVolume  = "nginx_content"
      containerPath = "/usr/share/nginx/html"
      readOnly      = false
    }
  ]
  volumes = [
    {
      host_path = "/efs/nginx_content"
      name      = "nginx_content"
      docker_volume_configuration = []
    }
  ]
  service_desired_count = 2
  port_mappings = [
    {
      containerPort = 80
      hostPort = 8080
      protocol = "tcp"
    }
  ]
  target_groups = [
    {
      hostPort = 8080
      target_group_arn = "arn:aws:elasticloadbalancing:us-east-2:888888888888:targetgroup/my-web-server/b8fbca622c86d2dd"
    }
  ]
  deploy_with_tg = true
}

```

## Deploying a service wihtout a Load Balancer
```hcl
module "task_without_alb" {
  source  = "lazzurs/ecs-service/aws"
  version = "0.4.0"  
  ecs_cluster_id                = "arn:aws:ecs:us-east-1:888888888888:cluster/ecs-0"
  service_name                  = "datadog_agent"
  image_name                    = "datadog/agent:latest"
  service_cpu                   = 10
  service_memory                = 256
  essential                     = true
  mount_points                  = [
        {
          containerPath = "/var/run/docker.sock"
          sourceVolume = "docker_sock"
          readOnly = true
        },
        {
          containerPath = "/host/sys/fs/cgroup"
          sourceVolume = "cgroup"
          readOnly = true
        },
        {
          containerPath = "/host/proc"
          sourceVolume = "proc"
          readOnly = true
        }
  ]
  environment                   = [
        {
          name = "DD_API_KEY"
          value = "55555555555555555555555555555555"
        },
        {
          name = "DD_SITE"
          value = "datadoghq.com"
        }
  ]
  volumes                       =  [
    {
      host_path = "/var/run/docker.sock"
      name      = "docker_sock"
      docker_volume_configuration = []
    },
    {
      host_path = "/proc/"
      name      = "proc"
      docker_volume_configuration = []
    },
    {
      host_path = "/sys/fs/cgroup/"
      name      = "cgroup"
      docker_volume_configuration = []
    }
  ]
}
```

## Variables
| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| ecs_cluster_id | ID of the ECS cluster | string | | yes |
| service_name | Name of the service being deployed | string | | yes |
| image_name | Name of the image to be deployed | string | | yes |
| port_mappings | Port mappings for the docker container | list(object) | [] | no |
| target_groups | Target group mappings for the docker container | list(object) | [] | no |
| mount_points | Mount points for the container | list | [] | no |
| environment | Environmental variables to pass to the container | list | [] | no | 
| linux_parameters | Additional Linux Parameters | object | null | no |
| service_desired_count | Desired number of instances to run | number | 1 | no |
| service_cpu | CPU units to allocate | number | 128 | no |
| service_memory | Memory to allocate | number | 1024 | no |
| vpc_name | VPC that the service is deployed in | string |  "" | no |
| tld | Top Level Domain to Use | string | "" | no |
| health_check_path | Health check path for the ALB | string | "/" | no |
| volumes | Task Volume definitions as a list of configuration objects | list(object) | [] | no|
| tags | A map of tags to add to all resources | map(string) | {} | no |
| create_listener | Create the alb listener (only needed once per port) | bool | false | no |
| task_iam_policies | Additional task policies to be applied | list(object) | [] | no |
| essential | Whether the task is essential | bool | true | no |
| privileged | Whether the task is privileged | bool | false | no |
| command | The command that is passed to the container | list(string) | [] | no |
| network_mode | The Network Mode to run the container at | string | bridge | no | 
| log_configuration | Log configuration options to send to a custom log driver for the container | object | null | no |
| network_configuration | Network configuration for awsvpc networking type | object | [] | no |
| deploy_with_tg | Deploy the service group attached to a target group | bool | false | no |
| dns_search_domains | DNS search domains to use for DNS lookups | list(string) | no | [""] |


## Outputs

None.

## Authors
Module is forked from a module by [Mark Honomichl](https://github.com/austincloudguru).
Maintained by Rob Lazzurs

## License
MIT Licensed.  See LICENSE for full details
