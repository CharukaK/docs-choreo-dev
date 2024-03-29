# Autoscale Components with Scale to Zero

Choreo provides the scale-to-zero capability for HTTP applications you deploy in the data plane. This lets you run your components in a serverless mode.

When this feature is active, your apps will automatically scale down to zero unless they receive HTTP traffic. When the application receives an HTTP request, your workload quickly scales up from zero to handle the request.

Scale to zero is very useful in lower environments, where you can significantly reduce infrastructure costs by scaling down idle workloads. In production environments, you can also use scale-to-zero capability if your application's behavior aligns with this feature behavior. In the paid tier, if you want to run your application with more guaranteed high availability, it is recommended to choose HPA (Horizontal Pod Autoscaler) scaling method and configure a minimum replica count of 2 or higher.

## Enable scale to zero

For service components and web-apps you create after February 23, 2024, Choreo enables the scale-to-zero feature by default. When deploying or promoting the component, the deployment will automatically scale-to-zero.
Upon the next request to the deployed service, a replica will be created to serve the request.

!!! note  
    - For the services which contain at least one endpoint with the network visibility as **Project**, Choreo will not automatically scale-to-zero those components when you deploy or promote them.
    - HTTP services that run on a port other than the below list of ports will not automatically scale-to-zero your component when deploying or promoting: 5000, 6000, 7000, 8000, 9000, 7070 to 7079, 8080 to 8089, and 9090 to 9099 or 8290.


To enable scale-to-zero for service components created before February 23, 2024, follow the steps given below:

1. Sign in to the [Choreo Console](https://console.choreo.dev/).
2. In the **Component Listing** pane, click on the component you want to scale-to-zero. 
3. Make sure the component is deployed to an environment and is ready to receive traffic.
4. In the left navigation menu, click **DevOps** and then click **Scaling**.

- **If you are a free user**, you will see a view similar to the one below. You can click the **scale-to-zero** card to enable scale-to-zero for your component.

![Free User - Scale to Zero](../../assets/img/devops-and-ci-cd/scaling/free-user-scaling-view.png){.cInlineImage-full}

- **If you are a paid user or you are running your applications in your own private data plane**, you will see a view similar to the one below. You can click the **scale-to-zero** card to enable scale-to-zero for your component.

![Paid User - Scale to Zero](../../assets/img/devops-and-ci-cd/scaling/paid-user-scaling-view.png){.cInlineImage-full}

You can independently scale Choreo components in both the **Development** and **Production** environments. The deployment card indicates the scaling status of each environment. To configure the scale-to-zero feature for a specific environment, click on the **scale-to-zero** link, which redirects to the **Devops** → **Scaling** page.

![Deploy View - Scale to Zero](../../assets/img/devops-and-ci-cd/scaling/scale-to-zero-in-deploy-view.png){.cInlineImage-full}

When you turn on the scale-to-zero for your application, the minimum replicas for your app will be set to zero. However, you can still select an appropriate maximum number of replicas.

## Limitations

- The scale-to-zero feature currently exclusively supports web applications and HTTP services. TCP and HTTPS services are not supported to be scaled to zero.
- To scale to zero, your HTTP service must run on one of the specified ports: 5000, 6000, 7000, 8000, 9000, 7070 to 7079, 8080 to 8089, and 9090 to 9099 or 8290. If you have an endpoint in your component running in any other port, your component will not automatically scale-to-zero when deploying or promoting. Also, if you try to switch to the “scale-to-zero” option in the “Devops” → “Scaling” view, it will fail.
- Currently, HTTP services which have “Project” network visible endpoints are not supported with scale-to-zero.
  If you have such an endpoint in your component, your component will not automatically scale-to-zero when deploying or promoting. Also, if you try to switch to the “scale-to-zero” option in the “Devops” → “Scaling” view, it will fail.

## Architecture 

When your Choreo application scales down to zero, an intermediary proxy service intercepts incoming requests. If a request is directed at your application, this service initiates a scale-up. Requests are held in the proxy's queue until your application becomes active. After scaling up, the proxy forwards the queued requests to your application.

If your application remains without HTTP traffic for an extended period (default idle time is 5 minutes), it will be scaled down to zero until more HTTP requests arrive. Conversely, if there's a surge in HTTP traffic to your scaled-up application, Choreo will further increase its scale to manage the demand. Choreo considers adding additional replicas if the number of queued requests surpasses the 'Target Pending Requests' threshold, which is set to 100 by default. You can adjust this threshold in the user interface.

!!! note 
    The initial request after a long period of inactivity experiences a delay because the application must first scale up from zero. If your API operates in a service-chain sequence (e.g., service-1 activates service-2, which in turn calls service-3), this waiting time may extend further. If your application or its chain takes a considerable time to scale up, be aware that the first request might face a timeout.

## Troubleshooting

When Choreo enables scale-to-zero by default, it will configure the readiness probe with some default values. However, in some cases, you may observe that your first request responds with a 503 status code. To overcome these behaviors, fine-tune the readiness probe in the **DevOps** → **Health Checks** view to match your application's needs.
