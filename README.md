﻿# Design_the_iac
 provider "aws" {
  region = "us-east-2"
  secret_key = "Dnrz9GyeIRrjWmFNqFcufAU/XFQHN/cS/6IJjlWW"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.1.0/24"
  enable_dns_support = true
  enable_dns_hostnames = true
}
 


resource "aws_subnet" "subnet" {
  vpc_id            =  "vpc-0f6fdd80d3308af8f"
  cidr_block        = "10.0.0.0/24"
  availability_zone = "us-east-2b"
}

resource "aws_security_group" "sg" {
  vpc_id =  aws_vpc.main.id
}
resource "aws_ecs_cluster" "cluster" {
  name = "medusa-cluster"
}
resource "aws_ecs_task_definition" "medusa" {
  family                   = "medusa-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "512"
  memory                   = "1024"

  container_definitions = jsonencode([{
    name      = "medusa"
    image     = "your-ecr-repo/medusa:latest"
    essential = true
    portMappings = [
      {
        containerPort = 9000
        hostPort      = 9000
        protocol      = "tcp"
      }
    ]
  }])
}
resource "aws_ecs_service" "medusa" {
  name            = "medusa-service"
  cluster         = aws_ecs_cluster.cluster.id
  task_definition = aws_ecs_task_definition.medusa.arn
  desired_count   = 1
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = [aws_subnet.subnet.id]
    security_groups  = [aws_security_group.sg.id]
    assign_public_ip = true
  }

  depends_on = [
    aws_lb_listener.front_end
  ]
}
resource "aws_lb" "front_end" {
  name               = "my-load-balancer"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.sg.id]
  
}
output "subnet_id" {
  value = aws_subnet.my_subnet.id
}
