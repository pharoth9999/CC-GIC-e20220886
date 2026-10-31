# Cloud Lab 2 Submission — Load Balancing and Auto Scaling

- **Student name:** THY PHAROTH
- **Student ID:** e20220886
- **Learner Lab user name:** user5623384
- **Date completed:** 2026-10-30

---

## Task evidence

### Load balancer security group

![Load balancer security group](sreen_shot/01-sg-load-balancer.png)

`LoadBalancerSecurityGroup` (sg-028e8854c5243cb3b) was created successfully. It has one inbound rule that allows HTTP (TCP 80), so the load balancer can receive web traffic from the internet.

### Launch template security group

![Launch template security group](sreen_shot/02-sg-launch-template.png)

`LaunchTemplateSecurityGroup` (sg-0ba27c81f40c2cb3e) was created with two inbound rules: SSH (TCP 22) and HTTP (TCP 80). The instances launched by Auto Scaling use this group.

### Target group

![Target group](sreen_shot/03-target-group.png)

`LoadBalancerTargetGroup` was created with target type **Instance** and protocol **HTTP:80**. At this point it has 0 targets and no load balancer, which is expected because the Auto Scaling group has not been created yet.

### Health checks

![Health check settings](sreen_shot/04-health-checks.png)

The health check sends HTTP requests to path `/` on the traffic port every 30 seconds and expects code `200`. A target becomes healthy after 5 successes in a row and unhealthy after 2 failures in a row.

### Load balancer

![Load balancer list](sreen_shot/05-load-balancer-list.png)

`MyLoadBalancer` is an **Application** load balancer. It is **Internet-facing** and its status is **Active**. It covers six Availability Zones (us-east-1a to us-east-1f) and its DNS name is `MyLoadBalancer-2142547702.us-east-1.elb.amazonaws.com`.

![Load balancer details — security](sreen_shot/06-load-balancer-details.png)

The **Security** tab shows that the load balancer uses `LoadBalancerSecurityGroup` (sg-028e8854c5243cb3b), the HTTP-only group created earlier.

### Launch template

![Launch template](sreen_shot/07-launch-template.png)

`AutoScalingLaunchTemplate` (lt-08b2735b07657fb27) launches **t2.micro** instances from ami-073f45061d35c86f0, with key pair `vockey` and security group sg-0ba27c81f40c2cb3e.

### Auto Scaling group

![Auto Scaling group list](sreen_shot/08-asg-list.png)

`MyAutoScalingGroup` was created from `AutoScalingLaunchTemplate` with desired capacity 2. Its status is **Updating capacity** because the first instances are still launching.

![Auto Scaling group details](sreen_shot/09-asg-details.png)

The group has **desired capacity 2** and **scaling limits 2 – 3**, and uses the default version of the launch template.

### Browser test through the load balancer

![Browser test 1](sreen_shot/10-browser-1.png)

![Browser test 2](sreen_shot/11-browser-2.png)

Opening the load balancer DNS name in a browser returns `Hello World from (ip-172-31-0-131.ec2.internal)`. This proves that the load balancer forwards requests to an instance in the Auto Scaling group.

### Stopping the web server

![Stop httpd](sreen_shot/12-stop-httpd.png)

On instance i-060c17c8c4527fbca (ip-172-31-0-131), I ran `sudo systemctl stop httpd.service`. `systemctl status httpd` then shows the service as **inactive (dead)**, so this instance no longer serves the web page.

```bash
sudo systemctl stop httpd.service
systemctl status httpd
```

### Target marked unhealthy

![Target unhealthy](sreen_shot/13-target-unhealthy.png)

After a few failed health checks, the target group shows 1 healthy target (i-01f0fb7dfc4dd3d3f) and 1 **unhealthy** target (i-060c17c8c4527fbca, "Health checks failed").

### Unhealthy instance replaced

![Target replaced](sreen_shot/14-target-replaced.png)

Auto Scaling responds to the failed instance automatically:

- The old instance i-060c17c8c4527fbca is **Draining**, which means it is being deregistered.
- A new instance i-078b9ab4e7e175182 has been launched in us-east-1b to replace it.
- The new instance still shows "Health checks failed" because it is booting. It turns healthy after it passes 5 checks in a row.

---

## Questions

<!-- Answer the lab's questions here, in your own words. -->

1. **Question 1**

   Answer.

2. **Question 2**

   Answer.

---

## Problems and how I solved them (optional)

<!-- An error you met, what caused it, and how you fixed it. -->

## AI note (optional)

<!-- One suggestion an AI tool gave you and how you checked it. -->

## Clean up

- [ ] Auto Scaling group `MyAutoScalingGroup` deleted (this also terminates its instances).
- [ ] Load balancer `MyLoadBalancer` deleted.
- [ ] Target group `LoadBalancerTargetGroup` deleted.
- [ ] Launch template `AutoScalingLaunchTemplate` deleted.
- [ ] Security groups `LaunchTemplateSecurityGroup` and `LoadBalancerSecurityGroup` deleted.
