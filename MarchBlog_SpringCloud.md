# Managed Virtual Network and Autoscale are now generally available in Azure Spring Cloud

We are excited to announce the general availability of the Azure Spring Cloud Managed Virtual Network and Autoscale features. Both are critical to help securely run production workloads at scale on the Azure Spring Cloud service. 

## Secure Azure Spring Cloud in Managed Virtual Network
With the Managed Virtual Network feature, you can provision the Azure Spring Cloud service in your  virtual network which enables:

-	Isolation of Azure Spring Cloud apps and service runtime from the internet on your corporate network.

-	 Interacting with systems in on-premises data centers or Azure services in other virtual networks.

-	Controlling inbound and outbound network communications for Azure Spring Cloud.

Also, we’ve enabled bringing your own route table for custom route management. You can supply subnets that come with attached route tables with UDRs (User-Defined Routes) that govern access to your on-premises system, and outbound internet connections. 

For example,  if your custom subnet contains a route table when you create an Azure Spring Cloud service instance, Azure Spring Cloud acknowledges the existing route table during service creation and adds/updates rules accordingly. If your custom subnet does not contain a route table, Azure Spring Cloud creates one for you and adds rules to it throughout the service lifecycle. 

![Figure 1](https://github.com/kyliel/AzureSpringCloudBlog/blob/master/Images-vnet-autoscaler-ga_202103/Figure1_VNET.png) 
<span style="color:gray">*Figure 1: Azure Spring Cloud in a managed virtual network with custom route table*</span>.


Azure Spring Cloud also provides self-diagnostics to help you troubleshoot networking connectivity issues such as misconfiguring private DNS zones. 
 
![Figure 2](https://github.com/kyliel/AzureSpringCloudBlog/blob/master/Images-vnet-autoscaler-ga_202103/Figure2_VNET.PNG) 
<span style="color:gray">*Figure 2: Diagnostics page to diagnose issue such as networking connectivity*</span>.

![Figure 3](https://github.com/kyliel/AzureSpringCloudBlog/blob/master/Images-vnet-autoscaler-ga_202103/Figure3_VNET.png)  
<span style="color:gray">*Figure 3: Examples of DNS Resolution self-diagnostics*</span>.

To learn more, get started with [Deploy Azure Spring Cloud in a Virtual Network](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-deploy-in-azure-virtual-network). 

## Dynamically Scale your Azure Spring Cloud Apps to meet changing demands

With the Azure Spring Cloud Autoscale feature, you can automate the upscaling or downscaling of the application to meet demand at peak times, and scale back when it is not necessary to reduce operational cost. Once Autoscale is enabled, the service will take care of your underlying infrastructure, and the load on your apps.

![Figure 4](https://github.com/kyliel/AzureSpringCloudBlog/blob/master/Images-vnet-autoscaler-ga_202103/Figure4_Scale.png) 
<span style="color:gray">*Figure 4: Metrics with upscaling and downscaling* </span>.

As some apps are CPU-bound, and others are memory-bound, you can pick up the metric and define scaling rules based on its value. If your traffic always skyrockets at 9am Monday through Friday, you can schedule more aggressive autoscaling targets for the work week.

In Azure Spring Cloud, an App is an abstraction of one business app or one microservice. One version of code or binary deployed as the App runs in a Deployment. You can have one active deployment for production and the other deployment for staging. You can configure Autoscale setting for each deployment.

![Figure 5](https://github.com/kyliel/AzureSpringCloudBlog/blob/master/Images-vnet-autoscaler-ga_202103/Figure5_Scale.PNG)  
<span style="color:gray">*Figure 5: Autoscale settings of Azure Spring Cloud App/Deployment*</span>.

You can learn how to [set up Autoscale for Azure Spring Cloud Apps/Deployments](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-setup-autoscale). And you also can automate Autoscaling Azure Spring Cloud Apps/Deployments with Terraform. Below is an example. 

```Terraform
resource "azurerm_monitor_autoscale_setting" "test" {
  name                = "acctestautoscale-cz"
  resource_group_name = data.azurerm_resource_group.test.name
  location            = data.azurerm_resource_group.test.location
  target_resource_id  = azurerm_spring_cloud_java_deployment.test.id
  enabled             = true
  profile {
    name = "metricRules"
    capacity {
      default = 1
      minimum = 1
      maximum = 2
    }
    rule {
      metric_trigger {
        dimensions {
          name     = "AppName"
          operator = "Equals"
          values   = [azurerm_spring_cloud_app.test.name]
        }

        dimensions {
          name     = "Deployment"
          operator = "Equals"
          values   = [azurerm_spring_cloud_java_deployment.test.name]
        }

        metric_name        = "AppCpuUsage"
        metric_namespace   = "microsoft.appplatform/spring"
        metric_resource_id = azurerm_spring_cloud_service.test.id
        time_grain         = "PT1M"
        statistic          = "Average"
        time_window        = "PT5M"
        time_aggregation   = "Average"
        operator           = "GreaterThan"
        threshold          = 75
      }
      scale_action {
        direction = "Increase"
        type      = "ChangeCount"
        value     = 1
        cooldown  = "PT1M"
      }
    }
  }
}
```

## Get Started
*	[Step by step tutorials](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart?tabs=Azure-CLI&pivots=programming-language-java): Learn the basics of Azure Spring Cloud with well-known Spring sample apps. 

*	[Online workshop](https://docs.microsoft.com/learn/modules/azure-spring-cloud-workshop/): Go through tasks to deploy Spring Boot microservices to Azure Spring Cloud with an Azure database for MySQL.

* [Troubleshooting tips](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-troubleshoot): Read common tips for troubleshooting Azure Spring Cloud server- and client-side issues.

We continue to release new features and your input has been instrumental in shaping them, so please **[contact us](AzureSpringCloud-Talk@service.microsoft.com)** if you have any feedback or questions. 
