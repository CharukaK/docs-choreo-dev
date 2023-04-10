# Deploy a Containerized Application as an Event-Triggered Integration

Event-triggered integrations can help automate business processes and workflows in response to specific events. With Choreo, you can deploy your containerized application as an event-triggered integration component, which makes your application deployment process efficient and reduces manual intervention.

Consider a scenario where a developer has an event-triggered integration and needs to deploy it as a containerized application. This tutorial walks you through the steps to accomplish this requirement.

In this tutorial, you will do the following:

- Create an event-triggered integration component in Choreo by connecting a GitHub repository with a sample containerized application developed using Python. 

	!!! info
    	    The sample application creates a [RabbitMQ](https://www.rabbitmq.com/) topic, listens to it and logs incoming messages.

- Deploy the integration component to the development environment in Choreo.
- Test the integration component.


!!! tip "Before you begin"
    - Fork the [choreo examples](https://github.com/wso2/choreo-examples) GitHub repository. For this tutorial, you need the [containerized-rabbitmq-listener](https://github.com/wso2/choreo-examples/tree/main/ipaas/containerized/containerized-rabbitmq-listener) in the **choreo-examples** repository.  
 
    - Use an existing RabbitMQ instance or start a new [RabbitMQ](https://www.rabbitmq.com/download.html) instance. The instance should be accessible via the internet. Obtain the `username`, `host`, `password`, and `vhost` from the RabbitMQ instance to use later as environment variables. 

!!! tip "RabbitMQ instance"
    You can use a service such as [CloudAMQP](https://www.cloudamqp.com/) to easily set up a RabbitMQ instance.
  
## Step 1: Create the integration component

Follow the steps given below to create the integration component:

1. Go to [https://console.choreo.dev/ipaas](https://console.choreo.dev/ipaas) and sign in to the Choreo Console.

2. On the **Home** page, click **+Create Project**.

3. Enter a unique name and a description for the project, and select a region.

4. Click **Create**.  

5. Go to the **Event-Triggered Integration** card, click **Create**, and enter the following information:

    | **Field**       | **Value**               |
    |-----------------|-------------------------|
    | **Name**        | `MQTT listener`           |
    | **Description** | `My sample listener`    |
    
6. Click **Next**.

7. To allow Choreo to access your GitHub account, click **Authorize with GitHub**. 

	!!! tip
    	    If you have not already connected your GitHub repository to Choreo, enter your GitHub credentials, and select the [choreo examples](https://github.com/wso2/choreo-examples) GitHub repository to install the [Choreo GitHub App](https://github.com/marketplace/choreo-apps).

8. In the **Connect Repository** dialog box, enter the following information:

    | **Field**             | **Description**                                   |
    |-----------------------|---------------------------------------------------|
    | **GitHub Account**    | Your account                                      |
    | **GitHub Repository** | `choreo-examples`                                 |
    | **Branch**            | `main`                                            |
    | **Build Preset**      | Click **Dockerfile**                        |
    | **Dockerfile Path**   | `ipaas/containerized/containerized-rabbitmq-listener/Dockerfile` |  
    | **Docker Context Path**   | `ipaas/containerized/containerized-rabbitmq-listener` | 

	!!! tip
    	    - **Build Preset** specifies the type of build to run depending on the implementation of the component. It converts the integration code into a Docker image that can run on Choreo cloud. If the integration is developed using [WSO2 Integration Studio](https://wso2.com/integration/integration-studio/), select **Micro Integrator** as the build preset. If the integration is developed using the [Ballerina language](https://ballerina.io), select **Ballerina** as the build preset. If the integration is developed as a containerized application, select **Dockerfile** as the build preset.

            - **Dockerfile Path** specifies the location of the Dockerfile to build the component. 

9.   Click **Create**. This creates the component and takes you to the **Deploy** view, where you can proceed to deploy the component.


## Step 2: Deploy
Before you deploy the component, you must follow the steps given below to assign appropriate RabbitMQ credentials to the environment variables:

1. In the **Deploy** view, go to the **Configs & Secrets** tab to apply environment-specific configuration files, environment variables, and other secret resources to containers.
2. Click **Create**.
3. In the **Mount a Configuration** form, select **ConfigMap** as the configuration type, **Environment Variables** as the mount type and click **Next**.
4. Enter a **Config Name**. Here, let's specify `environmentconf` as the **Config Name**.
5. Specify the environment variables as key-value pairs. You must use the values you obtain from your RabbitMQ instance as the values for the following keys: 

    | **Key**    |
    |------------|
    | HOST       | 
    | USERNAME   |
    | PASSWORD   | 
    | VHOST      | 

6. Click **Create**. Now you are ready to deploy the component to the developer environment. 

7. Go to the **Build and Deploy** tab and then click **Deploy Manually** button in the Build Area card.

    !!! info
        You have to deploy the component manually the first time. Subsequent deployments will occur automatically because automatic deployment is enabled by default.

After Choreo deploys the component, you can proceed to test it.

## Step 3: Test

To test the component, you must publish messages to a designated topic in your RabbitMQ server.

1. Go to the RabbitMQ Management Console and publish the following message to the **TestQueue** topic.

    ```
      Hello World
    ```
   Now you are ready to observe the logs and see the message received by the integration component.

    !!! info
         The containerized Python app creates the **TestQueue** in the RabbitMQ queues once it is connected successfully. If you cannot see this queue in the queue list of your RabbitMQ instance, the app is most likely not connected correctly.

2. Go to the **iPaaS** profile and click the `MQTT listener` component you created.

3. Click **Observe** on the left navigation, go to the **Logs** section, and observe the message received by the integration component from the RabbitMQ server. You will see a message similar to the following:

    !!! info
            It can take a few minutes for messages to appear in the **Logs** section. To view the latest logs, click **Load Latest Logs**.

      ![Message entry in logs](../../assets/img/tutorials/event-triggered-integration/containerized-rabbitmq-message-in-logs.png){.cInlineImage-full}

      This confirms that the event-triggered integration is functioning as expected.

    !!! tip
            The `Observe` view allows you to observe statistics related to your Choreo component. For more information on observing a Choreo component, see the [Observability overview](../../observe-and-analyze/observe/observability-overview.md).


Congratulations! You have successfully created an event-triggered integration in Choreo with an existing containerized integration, deployed the integration component, triggered an event, and tested the component.
