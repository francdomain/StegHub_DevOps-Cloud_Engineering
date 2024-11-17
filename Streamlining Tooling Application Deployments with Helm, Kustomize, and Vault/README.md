# Streamlining Tooling Application Deployments with Helm, Kustomize, and Vault

This project provides a practical guide to deploying and managing a "tooling application" (replace with your specific application context) in Kubernetes using a robust combination of Helm, Kustomize, and Vault.

### Key Goals:

- __Efficient Deployments:__ Leverage Helm for packaging and managing the complex Vault application, while using Kustomize to tailor configurations for dev, sit, and prod environments.
- __Secure Secrets Management:__ Integrate Vault to securely store and inject database credentials into the tooling application, eliminating the need for hardcoded secrets.

Based on your experience on previous projects, you should already have a grasp of experience using Helm, but you should also know that there are multiple choices available when it comes to deploying applications into Kubernetes, so Helm should not always be your default choice. It is important to be aware of the options available, know the pros and cons, and choose what works for your team or project.

Let's quickly discuss the different options, and make an informed decision before starting the actual work.

1. __Write YAML files and deploy with _kubectl_:__ - This is the easiest method where you write YAML for deployments, services, ingress, and all of that, and then deploy with `kubectl apply -f <manifest-file-path>`. This is usually the default way when getting started with Kubernetes or during development and exploration. However, it is not sufficient or reliable when it comes to managing the infrastructure in production. It is hard work to keep track of multiple manifest files. You can imagine what will be the fate of your project if there are tens or hundreds of micro-services/applications that need to be managed across multiple environments with this type of approach, you will end up in a [PEBKAC Situation](https://www.google.com/search?q=PEBKAC+situation&rlz=1C5CHFA_enGB766GB766&oq=PEBKAC+situation&aqs=chrome..69i57j33i160.1849j0j7&sourceid=chrome&ie=UTF-8)

2. __Use a templating engine like _HELM___ - You already know about Helm. Its value propositions are to install applications, manage the lifecycle of those applications across multiple environments, and customise applications through templating. Without going deeper into its obvious benefits, it also has its downsides too. for example;

    1. Helm only adds value when you install community components. Tools you can easily find on [artifacthub.io](https://artifacthub.io/) otherwise you need to write the manifest files anyway.

    2. Leads to too much logic in the templates (This is not good for inexperienced Kubernetes users, and a problem for hiring managers)

    3. Learning another DevOps tool. Always be careful about introducing yet another tool into your team/project. Similar to the second option, it slows down the project.

    4. Everyone’s Kubernetes cluster is different. Your needs are different. You might need to make a change to the way the chart is deployed – like changing a small piece of configuration in the templates folder. But, like a remote control, you’ve only got a limited set of controls which are exposed to you (the values that are exposed in the `values.yamlfile`). If you want to make any deeper changes to the Helm chart, you’ll need to fork it and change it yourself. (This is a bit like taking a remote control apart, and adding a new button!)

3. __Kustomize Overlays__ - To overcome the challenges of __Helm__ identified above, using a tool that is already embedded as part of __kubectl__ which you are already familiar with makes more sense for most use cases. You will get to see how Kustomize works shortly. For now, understand that another option is to use Overlays, instead of a templating engine. The downside to this is that it does not manage the lifecycle of your applications like Helm does. Therefore, you will need to use other methods alongside Kustomize for this.

Before we make a final decision on how we will realistically manage deployments into Kubernetes, let's see how Kustomize works.

