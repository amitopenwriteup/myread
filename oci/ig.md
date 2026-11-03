# Teleprompter Script: OCI Native Ingress Controller on OKE

Companion script for **OCI_Native_Ingress_Detailed_Lab.pptx** (28 slides).

**Total running time:** about 55 to 65 minutes with live demo, about 35 minutes slides only.
**Speaking pace:** about 140 words per minute. Times are per slide.

## How to use this script

- Read the **plain paragraphs** aloud. They are written to be spoken, so commands are described, not spelled out.
- Lines that start with **[PAUSE]** mean stop for two seconds and let the slide sink in.
- Lines that start with **[SHOW]** are actions on screen. Do not read them aloud.
- Lines that start with **[ASK]** are questions for the audience. Wait for an answer.
- Lines that start with **[TRAINER NOTE]** are for you only.
- Text in `code style` is something to say as a word, such as `kubectl`, not to spell out.

## Before you start (5 minutes, off camera)

- [ ] Cluster is `ACTIVE`, `kubectl get nodes` shows a `Ready` node.
- [ ] `COMPARTMENT_OCID`, `CLUSTER_OCID`, `LB_SUBNET_OCID` are set in your terminal.
- [ ] The load balancer subnet allows TCP 80 from your client (security list or NSG).
- [ ] Your jumpbox or laptop can reach the internet, so it can reach the load balancer public IP.
- [ ] cert-manager and the add-on are installed ahead of time if you want a short demo. Otherwise allow 10 extra minutes for the add-on install.
- [ ] Terminal font is large. Two windows ready: terminal and OCI Console.

---

## Slide 1: Title (30 seconds)

Welcome, everyone.

Today we are going to install the OCI Native Ingress Controller on an OKE cluster and use it to route traffic to several applications through one OCI load balancer.

This is a hands-on session. You will see the concepts first, then every lab step, including the problems people usually hit and how to fix them.

[PAUSE]

Let us start with the big picture.

---

## Slide 2: What you will build (1 minute)

[SHOW] Point along the flow diagram from left to right.

Here is what we are building. A client sends an HTTP request. It reaches one OCI load balancer. The load balancer sends it to the worker nodes, on a node port. From there it reaches the right pods.

Why does this matter? Normally, every Kubernetes Service of type LoadBalancer creates its own load balancer. Ten services means ten load balancers, and ten bills.

With Ingress, one load balancer serves many hosts and many paths.

The controller owns that load balancer. It creates and updates the listeners, backend sets and routing rules for you, based on your Ingress and IngressClass objects.

One warning now, so nobody forgets. This load balancer is billable. We will delete everything in the last lab.

---

## Slide 3: Lab roadmap (1 minute)

[SHOW] Slide with the ten labs.

There are ten labs.

Lab one is prerequisites and collecting the identifiers we need. Lab two prepares the API key and the authentication config. Lab three installs cert-manager first, then the controller as an OKE add-on.

Lab four deploys two sample apps. Lab five creates the IngressClass and its parameters. Lab six creates the Ingress with host and path routing.

Lab seven tests the routes. Lab eight is optional and adds TLS. Lab nine verifies everything in OCI. Lab ten cleans up.

Each lab has a clear finish line, so you always know whether you are ready for the next step.

---

## Slide 4: How the controller authenticates (1.5 minutes)

[SHOW] Walk through the four boxes.

The controller calls OCI APIs, so it needs an identity. In this guide it acts as an OCI user, using that user's API key.

The flow is simple. We take the API key details from the OCI CLI config. We build an auth config file. We store the config and the private key in a Kubernetes Secret named `oci-config`. The controller reads that Secret.

This needs no IAM changes of its own, but the user's group must already allow load balancers, certificates and networking. If something is missing, the controller logs a four-oh-one, four-oh-three or four-oh-four with the operation name.

[PAUSE]

Look at the orange box. An API private key goes into the cluster. That is fine for a short demo. For anything longer, use a dedicated user, and always delete the Secret and key copies at the end.

---

## Slide 5: Lab 1, Prerequisites and OCIDs (2 minutes)

[SHOW] Left side bullets, then the code on the right.

You need a cluster with a ready worker node and working `kubectl`. The Kubernetes version must be supported by the controller. At the time of writing the docs list one point twenty-seven through one point thirty-six, so check the current list.

Networking must be either VCN-native pod networking or Flannel, with security rules that let the load balancer reach the workers. You also need a load balancer subnet, and the OCI CLI with an API key.

On the right, we collect two identifiers. First the compartment, by listing compartments. Then the cluster OCID, by name, filtered to active clusters.

[SHOW] Run the `echo` of the cluster OCID.

The result must start with `ocid1.cluster.oc1`. If it prints null, the name or compartment is wrong, or the cluster is not active.

---

## Slide 6: Lab 1, Kubeconfig and load balancer subnet (2 minutes)

[SHOW] Code on the left, table on the right.

Next we create the kubeconfig and check the node. Then we list subnets and pick one for the load balancer.

The table shows how to choose. If the private column says false, it is a public subnet, for an internet-facing load balancer. If it says true, it is private, for an internal load balancer reachable only inside the VCN.

Two tips. Avoid the Kubernetes API endpoint subnet if you can. Its rules rarely allow ports eighty and four-forty-three, or access to the workers.

And echo both variables before moving on. An empty value later breaks the add-on with an error about value size between one and eighty-one ninety-one.

---

## Slide 7: Lab 2, Read values and check the key (2 minutes)

[SHOW] Code on the left, table on the right.

Here we read the user OCID, the fingerprint and the key file path from the OCI config, and look at the first line of the key.

That first line matters. The controller cannot use an encrypted private key. If the Secret holds one, the pod crashes with a message about failing to parse the private key.

[SHOW] Table on the right.

If the first line says encrypted private key, we decrypt it on the next slide. If it says RSA private key or just private key, use it as is. If it says OpenSSH or ssh-rsa, that is an SSH key, the wrong file. If it says public key, you picked the wrong file. Use the private one.

---

## Slide 8: Lab 2, Decrypt and create the auth config (2 minutes)

[SHOW] Two code boxes.

On the left, only if the key is encrypted, we make an unencrypted copy with `openssl`, restrict its permissions, and check the first line again. The fingerprint does not change, so nothing changes in OCI.

On the right, we create the auth config file. It holds the region, tenancy, compartment, user, fingerprint, and an empty passphrase, because the key is now unencrypted.

Before moving on, check two things. The first line of the key must not say encrypted. And every field in the file needs a value.

---

## Slide 9: Lab 2, Permissions the user needs (1.5 minutes)

[SHOW] Two cards, then the policy line at the bottom.

You only need this slide if the controller logs a four-oh-one, four-oh-three or four-oh-four.

On the left, the compartment-level permissions: manage load balancers, use virtual network family, manage the certificate resources, manage the web application firewall family, and read the cluster family.

On the right, three tenancy-level permissions: read public IPs, manage floating IPs, and use tag namespaces. These three are the ones most often missing.

At the bottom is one example statement. Note that when you use an OCID, the statement must say "in compartment id", followed by the OCID.

[TRAINER NOTE] Tell the audience to confirm the full list on Oracle's prerequisites page.

---

## Slide 10: Lab 3, cert-manager first (2 minutes)

[SHOW] Flow boxes, then the code.

Lab three has four steps, and the order matters. First cert-manager, then the add-on, then the Secret, then verify.

Why cert-manager first? The controller's webhook certificate comes from cert-manager. If it is missing when the controller is installed, the pod stays in ContainerCreating with an error saying the secret `oci-native-ingress-controller-tls` was not found.

[SHOW] Run the apply command and the wait command.

We apply the cert-manager manifest, watch the pods, and then wait until the deployments are available. Do not continue until this succeeds.

---

## Slide 11: Lab 3, Add-on configuration (1.5 minutes)

[SHOW] Bullets on the left, JSON on the right.

The add-on configuration is a small JSON file. It needs the compartment, the load balancer subnet, and two authentication settings: auth type set to user, and auth secret name set to `oci-config`.

Remember the key is `loadBalancerSubnetId`, not subnet ID.

[SHOW] Run the `cat` of the file.

Always print the file and check that no value is empty. That one check saves a lot of time.

---

## Slide 12: Lab 3, Install the add-on (1.5 minutes)

[SHOW] Run the install command.

We run `install-addon` with the JSON file. This creates a work request. We then list add-ons and wait for the lifecycle state to say ACTIVE.

The bottom of the slide also shows `update-addon`. One important detail: it replaces the whole configuration, so include every key, not only the one you are changing.

You can also install from the Console, under Developer Services, Kubernetes Clusters, and Add-ons.

---

## Slide 13: Lab 3, Auth Secret and verify (2 minutes)

[SHOW] Code on the left, cards on the right.

Now the Secret. It must be in the add-on's namespace, which exists once the add-on is installed. The Secret holds exactly two keys: `config` and `private-key`.

After creating it, we restart the controller deployment, then check the pods and the ingress class. The pod should be one of one, running, with a stable restart count.

[SHOW] Orange card.

Reading the log: you will see "failed to acquire lease" messages. That is normal for about a minute while a restarted pod waits for the old leader. After that you should see "successfully acquired lease" and a line about fetching the auth config provider for type user.

If you see a four-oh-one, four-oh-three or four-oh-four after that, it is a permission problem.

---

## Slide 14: Fix, missing TLS Secret (2 minutes)

[SHOW] Code box, then the table.

This is the most common install problem. The pod sits in ContainerCreating and `describe pod` shows a mount failure for the secret `oci-native-ingress-controller-tls`.

The cause is almost always cert-manager missing, or installed after the controller.

Three checks decide the fix. If there is no cert-manager namespace or CRDs, install it and wait for three pods. If cert-manager is running but there is no Certificate or Issuer, it was installed too late, so remove and reinstall the add-on. If the Certificate exists but is not Ready, describe the Certificate and CertificateRequest and read the events.

---

## Slide 15: Fix, remove and reinstall the add-on (1.5 minutes)

[SHOW] Bullets and code.

To reinstall, we disable the add-on with the remove-existing option, and list add-ons until it is gone.

Then we install again. If the namespace was removed, recreate the `oci-config` Secret.

Check that the TLS Secret now exists, then the pod. If the pod is still stuck, delete it and the Deployment recreates it.

[TRAINER NOTE] This is the slide to point back to whenever someone says "my pod is stuck".

---

## Slide 16: Lab 4, Sample applications (2 minutes)

[SHOW] Bullets and code.

We create a namespace called `ingress-demo` and two echo servers, each with two replicas. They report which pod answered.

Look at the Service type. It is NodePort, and this is important. On a Flannel cluster, the controller registers the worker nodes as load balancer backends on each Service's node port. If the Service is ClusterIP, the backend sets stay empty and the load balancer returns a five-oh-two.

[SHOW] Run the apply and get commands.

We should see four pods running, and each Service with a port like eighty, colon, three-something-thousand.

If you already created ClusterIP Services, the patch command at the bottom switches them in place.

---

## Slide 17: Lab 5, Class and parameters (2 minutes)

[SHOW] Two code boxes.

On the left, IngressClassParameters. This tells the controller how to build the load balancer: the compartment, subnet, load balancer name, whether it is private, and the bandwidth range.

On the right, the IngressClass. It links Ingress resources to the controller, and to the parameters object.

Two rules. Set `isPrivate` to match your subnet: false for public, true for private. And use the same subnet as the add-on's load balancer subnet.

If a default IngressClass already exists in the cluster, drop the default annotation here and set the class name on the Ingress instead.

---

## Slide 18: Lab 6, Ingress with host and path (2 minutes)

[SHOW] Bullets and code.

Now the Ingress. One host, `foo.bar.com`, and two paths. Slash testecho one goes to the first service. Slash testecho two goes to the second.

Both rules use path type Exact. Longest match wins, and Exact beats Prefix of the same length.

[SHOW] Run apply, then watch the Ingress.

The address stays empty at first. Usually one to three minutes after the controller becomes leader, the load balancer is created and the address appears. For a public load balancer it is a public IP. For an internal one, a private IP.

---

## Slide 19: Lab 6, Inspect and check for duplicates (1.5 minutes)

[SHOW] Code, then the three cards.

Use `describe ingress` and the events to see what the controller did.

Then a useful check. The controller log can briefly show a second request to create a load balancer. So confirm there is exactly one. Compare the load balancer list with the ID stored in the IngressClass annotation.

If you see two with the same name, keep the one whose ID matches the annotation. The other is an orphan, and you pay for it, so delete it.

Optional: for rolling updates, label the namespace for pod readiness gates before pods are created, or restart the deployments afterwards.

---

## Slide 20: Lab 7, Test the routes (3 minutes, live demo)

[SHOW] Run the commands in the terminal.

First we read the load balancer address from the Ingress status into a variable.

Then three requests. The first two use the Host header `foo.bar.com`, with the two paths. The Host header is what matches the rule. Each should return echo server output.

The third path has no rule, so we expect a four-oh-four or another non-two-hundred response.

[SHOW] Repeat one request a few times.

Run the same request again, and watch the pod name change. That is the load balancer spreading traffic across pods.

[TRAINER NOTE] If it hangs, use `curl -i -m 10` instead of `-s`. A timeout usually means the load balancer subnet does not allow port 80 from your client, or your machine has no internet route. A five-oh-two means no healthy backends: check NodePort Services and the worker security rules.

---

## Slide 21: Lab 8, Add TLS, optional (2 minutes)

[SHOW] Bullets and code.

This lab is optional. HTTPS terminates at the load balancer, using a Kubernetes TLS Secret.

We create a self-signed certificate for the demo, copy it to `ca.crt`, and create the Secret. It needs three keys: `ca.crt`, `tls.crt` and `tls.key`.

In the Ingress we set `backend-tls-enabled` to false, so the load balancer talks plain HTTP to our echo servers.

Then we call the HTTPS address. The `-k` flag accepts the self-signed certificate, because this is a demo.

An alternative is to point the Ingress at an existing OCI certificate with the `certificate-ocid` annotation.

---

## Slide 22: Lab 9, Verify in OCI (2.5 minutes, Console demo)

[SHOW] Open the OCI Console: Networking, Load balancers, then `native-ic-demo-lb`.

Here is what the controller built. Listeners: HTTP on port eighty, plus HTTPS if you did Lab 8. Backend sets: one per service, with your worker node IPs on the node ports. Routing policies: the host and path rules from your Ingress.

[SHOW] Run the health command.

This command shows overall health. Status should be OK once health checks pass. Set the load balancer OCID first.

[PAUSE]

Now read the orange box slowly. Do not edit this load balancer in the Console, CLI or API. The controller owns it and will revert your changes. Change the manifests and reapply.

---

## Slide 23: Demo run order (1 minute)

If you are presenting this lab to others, here is the order.

One: show the Ingress and its address. Two: run the two curl routes and show different apps answering. Three: show the same routes as listeners, backend sets and routing rules in the Console.

Four, the best part: add a third path to the Ingress, reapply, and watch the new rule appear in OCI. That proves the controller is reconciling for you.

---

## Slide 24: Lab 10, Clean up (2 minutes)

[SHOW] Steps on the left, code on the right.

Order matters here. Delete the Ingress resources first, so the controller removes the listeners and backend sets. Then the IngressClass, the apps, and the namespace.

Then remove the add-on, delete the `oci-config` Secret, and delete your key copies and config files. The decrypted key is an unprotected copy of your API key, so do not leave it behind.

Confirm in the Console that the load balancer is gone or deleting.

Also remember: certificates created from Secrets are not removed when an Ingress is deleted. Delete them in the OCI Certificates service. And remove cert-manager if you no longer need it.

---

## Slide 25: Troubleshooting, setup and auth (1.5 minutes)

[SHOW] Table.

Five common setup problems.

Cluster OCID empty: wrong name, wrong compartment, or the cluster is not active. Value size error: a configuration value is empty, often the subnet. Failed to parse private key: an encrypted or wrong key in the Secret.

Empty configuration or error retrieving secret: the `oci-config` Secret is missing, in the wrong namespace, or missing a key. And four-oh-one, four-oh-three, four-oh-four: a missing permission, or a wrong user, tenancy or fingerprint.

---

## Slide 26: Troubleshooting, controller (1.5 minutes)

[SHOW] Table.

Five controller problems.

Address stays empty: the controller is not leader, or load balancer creation failed. Check pod status and logs, including the previous run.

ContainerCreating with the TLS Secret missing: cert-manager was missing or late.

Repeated lease messages: wait a minute. If the holder keeps changing, the pod is crash-looping, so read the previous logs.

Ingress ignored or parameters not found: check the class name, and that the namespace and name match.

Two load balancers with the same name: keep the one that matches the IngressClass annotation.

---

## Slide 27: Troubleshooting, traffic (2 minutes)

[SHOW] Table.

Six traffic problems.

A four-oh-four: no rule matched. Check the Host header, and Exact versus Prefix. A timeout: security rules block your client, or you are calling an internal load balancer from outside the VCN.

A five-oh-two or five-oh-three with empty backend sets: the Services are ClusterIP on Flannel. Patch them to NodePort. If backends exist but health is critical, the load balancer cannot reach the workers. Allow the load balancer subnet to TCP thirty thousand through thirty-two seven-six-seven.

TLS errors: check the three keys in the Secret, and set backend TLS to false for plain HTTP backends.

And if your Console changes keep reverting, that is by design. The controller owns the load balancer.

---

## Slide 28: Key takeaways (1 minute)

Three things to remember.

One. One Ingress setup serves many hosts and paths from a single OCI load balancer.

Two. Order matters: cert-manager, then the add-on, then the `oci-config` Secret, then the class and Ingress. And use NodePort Services on Flannel.

Three. The controller owns the load balancer. Delete the Ingress resources first, then remove the Secret and key copies.

[PAUSE]

Thank you. Questions?

---

## Q and A cheat sheet

| Likely question | Short answer |
| --- | --- |
| Why not just use a LoadBalancer Service? | It creates one load balancer per service. Ingress shares one for many hosts and paths. |
| Can I use instance principal instead? | Yes. This guide uses a user and an API key, but the controller also supports instance principal and workload identity. Workload identity needs an Enhanced cluster. |
| Why NodePort Services? | On Flannel the controller registers worker nodes as backends on the node port. ClusterIP leaves the backend sets empty. |
| Why cert-manager? | The controller's webhook certificate comes from cert-manager. |
| Can I edit the load balancer in the Console? | No. The controller owns it and reverts changes. Change the manifests. |
| My curl times out. | Check the load balancer subnet allows port 80 from your client, that the load balancer is public, and that your machine has internet access. |
| My curl returns 502. | No healthy backends. Check NodePort Services, endpoints, and allow the load balancer subnet to the workers on TCP 30000 to 32767. |
| Does deleting the Ingress remove the certificate? | No. Certificates created from Secrets stay in OCI Certificates until you delete them. |
