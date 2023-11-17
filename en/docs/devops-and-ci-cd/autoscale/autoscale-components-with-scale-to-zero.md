# Autoscale Components with Scale to Zero
	
Choreo supports automatic scaling based on its usage. 

Choreo allows you enable scale-to-zero for your HTTP applications you deploy in the data plane. This lets you run your components in a serverless mode. When you enable this feature, your applications will automatically scale down to zero when not receiving HTTP traffic. In the event of an HTTP request, your workload rapidly scales up from zero to handle the request.

## Enable scale to zero

You can independently scale applications in both the `Development` and `Production` environments. Follow the steps below to enable scale-to-zero:

1. Go to the **Deploy** view of the component for which you want to enable scale to zero. Ensure the component is deployed to an environment and is ready to receive traffic. 
2. On the header, select the environment you want to apply scale to zero auto scaling from the **Environment** list.
2. In the left navigation, click **DevOps** and then click **Scaling**.
3. Click on the **Scale to Zero** toggle to enable it. 

Alternatively, you can enable the scale-to-zero feature for a specific environment, by clicking on the **Scale to Zero** link on the deployment card. It will redirect you to the **DevOps** -> **Scaling** section.

When you turn on the scale-to-zero option for your application using the UI toggle, the minimum replicas for your app will be set to zero. However, you can still pick an appropriate maximum number of replicas. 

The deployment card indicates the scaling status of each environment. 

## Limitations

- The scale-to-zero feature currently exclusively supports web applications and HTTP services.
- Your HTTP service must run on one of the specified ports: 5000, 6000, 7000, 8000, 9000, 7070 to 7079, 8080 to 8089, and 9091 to 9099.
- It is mandatory to set up readiness probes for your application through the “Devops” -> “Health Checks” page.
- When conducting intra-project service-to-service communication, remember to include the 'x-choreo-project-ns' header during a service call. Make sure the header's value aligns with your project's namespace name.

## Architecture 

When your Choreo application scales down to zero, an intermediary proxy service catches incoming requests. If a request is directed at your application, this service initiates a scale-up. Requests are held in the proxy's queue until your application becomes active. After scaling up, the proxy forwards the queued requests to your application. 

If your application goes without HTTP traffic for an extended period (default idle time is 5 minutes), it will be scaled down to zero until more HTTP requests arrive. Conversely, if there's a surge in HTTP traffic to your scaled-up application, Choreo will further increase its scale to manage the demand. Choreo considers adding additional replicas if the number of queued requests surpasses the 'Target Pending Requests' threshold, which is set to 100 by default. You can adjust this threshold in the user interface. 

Note that the initial request after a long period of inactivity will experience a delay because the application must first scale up from zero. If your API operates in a service-chain sequence (e.g., service-1 activates service-2, which in turn calls service-3), this wait time may extend further. If your application or its chain takes a considerable time to scale up, be aware that the first request might face a timeout.







