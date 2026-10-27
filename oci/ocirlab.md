# OCIR Labs, Parts 2, 3 and 4: Trainer Teleprompter Script

*[ACTION] cues are things you do on screen. Everything else is read aloud.*

---

## OPENING

Good morning, everyone. Welcome to the OCIR hands-on lab.

Today we'll work with Oracle Cloud Infrastructure Registry, or OCIR. It's a private place in Oracle Cloud to store your container images.

By the end of this session, you will have pushed an image, pulled it back, run it, managed versions, and cleaned everything up.

Before we start, make sure you have five things: an OCI tenancy and compartment, permission to manage repositories, Docker running, the OCI CLI configured, and an Auth Token.

If you're using Cloud Shell, Docker and the CLI are already installed. That is the easiest route.

---

## SETUP: NAMESPACE AND VARIABLES

First, find your tenancy namespace. Think of it as your tenancy's unique name inside the registry.

*[ACTION] Run: oci os ns get*

You'll see a short value in the output. Copy it.

Now we set environment variables so we don't retype long values. We set the region key, namespace, repository name, image name, and tag.

For Hyderabad, the region key is **hyd**, so our registry address is **hyd.ocir.io**. For any region, the format is region key, dot, ocir, dot, io.

---

## CLOUD SHELL: COMPARTMENT ID

Now an important point, because this is where most people get stuck.

The compartment ID must be a **compartment OCID**, starting with ocid1.compartment, or the **tenancy OCID**, starting with ocid1.tenancy.

Do **not** use an identity domain OCID. If you do, you'll get a DENIED error or "Invalid parameter compartmentId."

*[ACTION] Run the compartment list command with the subtree and table output flags.*

Copy the OCID of the compartment you want to use. If the list comes back empty, everything is in the root compartment, so use the tenancy OCID. In Cloud Shell, that's already stored in the variable OCI_TENANCY.

Next, set COMPARTMENT_ID, and set NAMESPACE with the command that fetches it automatically. Then echo the namespace to confirm it has a value.

---

## LIST AND CREATE THE REPOSITORY

Let's check what repositories already exist.

*[ACTION] Run: oci artifacts container repository list with the compartment ID.*

If the list is empty, that's fine. It just means we haven't created one yet.

If you see a permissions error, you need a policy like "Allow group, your group name, to manage repos in tenancy." Ask your admin.

Now we create the repository.

*[ACTION] Run the repository create command with display name myapp and is-public false.*

Note the flag **is-public false**. This makes the repository private. Anyone who wants to push or pull must log in with an Auth Token.

Run the list command again. You should now see myapp.

---

# PART 2: PUSH IMAGES TO OCIR

## Step 1: Build a local image

We need something to push. If you already have an image, skip this.

We'll create two small files. A Dockerfile that starts from nginx and copies in an index.html. And the index.html itself, which just says "Hello from OCIR."

*[ACTION] Run: docker build -t myapp:v1 .*

Then run **docker images** to confirm myapp v1 is there.

## Step 2: Log in to OCIR

Docker can't push to OCIR until it knows who you are. For that we use an **Auth Token**, not your console password.

To generate one: open your profile menu, choose My Profile, open Auth Tokens, and click Generate Token.

**Copy it immediately. It's shown only once.** Paste it somewhere safe.

*[ACTION] Run: docker login hyd.ocir.io*

When prompted for the username, enter your tenancy namespace, a slash, then your OCI username. For example: mytenancynamespace slash alice at example dot com.

For the password, paste the Auth Token.

If you are a federated user, your username also includes the identity domain: namespace, slash, domain, slash, username.

You should see "Login Succeeded."

## Step 3: Tag the image

Docker needs to know where this image should go. The OCIR image name follows one format: region key dot ocir dot io, slash namespace, slash repository, colon tag.

*[ACTION] Run docker tag, from myapp:v1 to the full OCIR path using our variables.*

Run docker images again. You'll see the same image with a second name. Same image, two labels.

## Step 4: Push

*[ACTION] Run: docker push hyd.ocir.io/$NAMESPACE/$REPOSITORY:$IMAGE_TAG*

Watch the layers upload. When it finishes, you'll see a **digest**, starting with sha256. That digest is the unique fingerprint of this image. Remember it. We'll use it later.

## Step 5: Verify in OCI

Let's confirm using the CLI.

*[ACTION] Run the repository list with display name myapp, then the container image list with repository name myapp.*

You should see your image with version v1.

---

# PART 3: PULL IMAGES FROM OCIR

## Step 1: Login

If you're still logged in from Part 2, skip this. Otherwise run docker login hyd.ocir.io again.

## Step 2: Pull

*[ACTION] Run: docker pull hyd.ocir.io/$NAMESPACE/$REPOSITORY:$IMAGE_TAG*

Since the image already exists locally, Docker may say it's up to date. For a true test, remove the local copy first, then pull.

## Step 3: Verify

Run docker images. You should see hyd.ocir.io, your namespace, myapp, v1.

## Step 4: Run it

*[ACTION] Run docker run with -d, the name myapp-container, and port 8080 mapped to 80.*

This runs nginx in the background and maps port 8080 on our machine to port 80 in the container.

Test it with curl on localhost 8080, or open it in a browser. You should see **"Hello from OCIR."**

That's the full loop: build, push, pull, run.

## Step 5: Pull by digest

Here's an important concept. A **tag** like v1 can be moved to point to a different image later. A **digest** can't. It always points to the exact same image.

So in production, pull by digest when you need certainty.

*[ACTION] Run docker pull with an at-sign and sha256 digest in place of the colon and tag.*

---

# PART 4: IMAGE MANAGEMENT TASKS

## Task 1: List repositories

*[ACTION] Run the repository list command.*

To see child compartments, adjust the compartment scope and make sure you have permissions there.

## Task 2: List images in a repository

*[ACTION] Run the image list command with the repository name.*

You can also write the repository as namespace slash repository name. Both work.

## Task 3: Push another version

*[ACTION] Tag myapp v1 as v2, then push v2.*

Now list the images again. You should see **both v1 and v2**.

## Task 4: Delete an image by OCID

First, list the images in table format and copy the **OCID** of the image to delete.

*[ACTION] Run: oci artifacts container image delete with --image-id.*

It will ask for confirmation. In a lab, you can add **--force** to skip that.

**Warning:** deleting an image is permanent. Double-check the OCID before you press Enter.

## Task 5: Delete by tag

Depending on your CLI version, you can delete using the repository name, image version, and compartment ID.

If your version doesn't accept that syntax, no problem. Use the OCID method from Task 4.

## Task 6: Delete the repository

This is our cleanup step.

*[ACTION] Run: oci artifacts container repository delete with --repository-id.*

Add --force for non-interactive deletion.

**Warning:** this removes all images in the repository. Only do this at the very end.

---

## VALIDATION CHECKLIST

Let's confirm you can do all of these:

Log in using an Auth Token. Tag an image with the correct OCIR path. Push it. List the images. Pull it. Run it locally. Push a second tag. Find an image OCID. Delete an image. And finally, delete the repository.

If you ticked all ten, you've completed the lab.

---

## COMMON MISTAKES: SAY THIS SLOWLY

Three mistakes cause most of the errors I see.

One: using the **tenancy OCID** instead of the **tenancy namespace** in the image path or username.

Two: using the **wrong region key**. Hyderabad is hyd.

Three: using your **OCI password** instead of the **Auth Token**.

If something fails, check these three first.

---

## CLOSING

Great work, everyone. You now know how to push, pull, and manage container images in OCIR. Any questions before we wrap up?

---

I can turn this into a Word document or slide notes with larger text for a teleprompter app, or add timing marks for each section.
