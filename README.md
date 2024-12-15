# MaskAnyone - Production Setup
This document describes the process of setting up a production environment for MaskAnyone.

## Prerequisites
Make sure you have installed [Docker](https://docs.docker.com/get-docker/) on your system and set the appropriate permissions. 
Furthermore, please also ensure that you have sufficient hard drive space available for the setup.
We recommend at least 50GB of free space.

## Setup
There are two different setup options. 

If you want to run MaskAnyone locally on your computer, please use the "Setup (Local)" option.
The functionality is generally the same, but you won't have a login and authentication service as you're the only person using it. 
This reduces complexity and setup duration a bit.

If you want to run MaskAnyone on a server where many different people can access it through the network, please use the "Setup (Main Server)" option.
This version includes an authentication service that allows you to configure user accounts to manage user access and separate user data.

## Setup (Local)

**Step 1: Open a terminal on your computer.**
This setup assumes that you're using a Linux(i.e. Debian)-based system. 
However, generally speaking, Mask Anyone should also run on any other common operating system by following this setup.

**Step 2: Pull this repository.**
Run the following commands one by one to pull the repository.
```bash
git clone https://github.com/MaskAnyone/MaskAnyoneProdInfrastructure.git maskanyone
cd maskanyone
```

**Step 3: Configure the environment variables.**
Now we need to set up the environment variables. First run the following command:
```bash
cp .env.dist .env
```
Now open the newly created `.env` file (e.g. using `nano .env`) and do the following adjustments:
- Replace `<your-strong-password-1>` with as password of your choice. This is the password used to access the database.
- (Optional) Configure the number of workers you want as well as their available resources. This is very dependent on the hardware resources of your computer and can be flexibly adjusted later on.

**Step 4: Pull the infrastructure.**
Run the following command to pull all the images and prepare the application infrastructure:
```bash
docker compose -f docker-compose-local.yml pull
```

**Step 5: Start the application for the first time.**
To do so first start the database container.
```bash
docker compose -f docker-compose-local.yml up -d postgres
```
Then wait for 10 seconds to give it some time to prepare the database. 
Afterward, also start the other containers.
```bash
docker compose -f docker-compose-local.yml up -d
```

> If you encounter an error like `Cannot start service proxy: driver failed programming external connectivity on endpoint mask-anyone-prod-test_proxy_1`, this likely means that another application is occupying the port 443. Stop this application and try again.

**Step 6: Verify that MaskAnyone is running.**
First check that all containers are up and running:
```bash
docker compose -f docker-compose-local.yml ps
```
If you find that a container is not running, you can try to identify the issue by looking at its logs:
```bash
docker compose -f docker-compose-local.yml logs -f <container-name>
```
If your containers are running, then please try accessing MaskAnyone at [https://localhost](https://localhost). 


## Setup (Main Server)

**Step 1: Open a terminal on your server.**
This can, for example, be done via ssh. Make sure you have the necessary permissions to execute the following commands.
This setup assumes that you're using a Linux(i.e. Debian)-based system. 
However, generally speaking, Mask Anyone should also run on any other common operating system by following this setup.

**Step 2: Pull this repository.**
Run the following commands one by one to pull the repository.
```bash
git clone https://github.com/MaskAnyone/MaskAnyoneProdInfrastructure.git maskanyone
cd maskanyone
```

**Step 3: Configure the environment variables.**
Now we need to set up the environment variables. First run the following command:
```bash
cp .env.dist .env
```
Now open the newly created `.env` file (e.g. using `nano .env`) and do the following adjustments:
- Replace `<your-strong-password-1>` with as password of your choice. This is the password used to access the database, so it is recommended to choose a strong password. Consider using a password generator (e.g. ``pwgen -cnys -r \`\" 32`` on Linux).
- Replace `<your-strong-password-2>` with as password of your choice. This is the password used to access the Keycloak admin interface (i.e. user management). Ideally, this should be different from the previous password.
- Configure the number of workers you want as well as their available resources. This is very dependent on the hardware resources of your server and can be flexibly adjusted later on.

> Please note: If you're setting up MaskAnyone on a server with the plan of accessing it from another computer, you may want to adjust some additional configurations. If you want to use a different port than the default https port (443), you can modify the `docker-compose-server.yml` and change the line `- "443:443"` in the `proxy` section to e.g., `- "1234:443"`. If you do that or plan on accessing MaskAnyone via the network (e.g., on `https://100.100.100.100:1234`), please also modify the `backend` section to specify an additional environment variable for the token issuer as follows: `BACKEND_AUtH_ISSUER=https://100.100.100.100:1234/auth/realms/maskanyone`.

**Step 4: Pull the infrastructure.**
Run the following command to pull all the images and prepare the application infrastructure:
```bash
docker compose -f docker-compose-server.yml pull
```

**Step 5: Start the application for the first time.**
To do so first start the database container.
```bash
docker compose -f docker-compose-server.yml up -d postgres
```
Then wait for 10 seconds to give it some time to prepare the database. 
Afterward, also start the other containers, except for the workers.
```bash
docker compose -f docker-compose-server.yml up -d proxy frontend backend keycloak
```
Then wait for another 30 seconds to give keycloak some time to complete its initial setup.
Now try to access the keycloak admin UI at [https://localhost/auth/](https://localhost/auth/). 
Remember that localhost will only work if the browser runs on the same computer where you're doing the setup.
If you're doing the setup on a server via SSH, then you can try accessing it using `https://<your-server-ip>/auth/`.

> If you encounter an error like `Cannot start service proxy: driver failed programming external connectivity on endpoint mask-anyone-prod-test_proxy_1`, this likely means that another application is occupying the port 443. Stop this application and try again.

**Step 6: Set up the keycloak realm.**
A realm is essentially an organization within keycloak where all your users and configuration is located. 
To create it please do the following:
- Open the keycloak admin UI
- Click on "Administration Console"
- Log in using `admin` and the password you specified
- In the top left corner you should see a select input, click on it and then click on "Create realm"
- Copy the [realm.json](realm.json) configuration file and paste it in the "Resource file" input field
- Click on "Create" to create your realm

**Step 7: Create a user**
At this point we can now create the first user with access to the MaskAnyone application. 
Follow these steps to set up a new user:
- Open the keycloak admin UI and login (if you haven't already)
- Make sure to select the "maskanyone" realm on the select in the top left corner
- Click on "Users" in the left sidebar (here you can now see all existing users if there are any)
- Click on "Add user"
- Specify a "Username", "Email", "First name" and "Last name"; Also set "Email verified" to true
- Click on "Create" to create this user
- Open the "Credentials" tab and click on "Set password" to specify a password for this user
- Specify the password and confirm it a second time; You can choose to leave temporary enabled which will force this user to change the password after the first login; Finally click on "Save" and "Save password"

**Step 8: Provide the backend with keycloak's public key.**
To identify a user the keycloak server generates a so-called token which it signs so that other parts of the application (i.e. our backend) can check the user permissions.
For this to work, however, the backend needs access to keycloak's public key. 
In order to give it access to this public key, please do the following:
- Open the keycloak admin UI, login and select the "maskanyone" realm (if you haven't already)
- Click on "Realm settings" in the left sidebar
- Open the "Keys" tab
- Search for the line with algorithm "RS256" and click on the "Public key" button of this row
- Copy the entire public key string shown in the dialog
- Open the `.env` file and replace `<keycloak-rs256-public-key>` with this key

> If you plan on accessing MaskAnyone on any other domain than `https://localhost` (so either a different port or hostname or both), then you'll also need to configure the root URL and valid redirect URIs of the keycloak client. To do so, please select the "maskanyone" realm, click on "Clients" and then "maskanyone-fe". Now please change the "Root URL" to e.g., `https://100.100.100.100:1234` and the first entry of "Valid redirect URIs" to `https://100.100.100.100:1234/*`.

**Step 8: Start the remaining containers.**
```bash
docker compose -f docker-compose-server.yml up -d
```

**Step 9: Verify that MaskAnyone is running.**
First check that all containers are up and running:
```bash
docker compose -f docker-compose-server.yml ps
```
If you find that a container is not running, you can try to identify the issue by looking at its logs:
```bash
docker compose -f docker-compose-server.yml logs -f <container-name>
```
If your containers are running, then please try accessing MaskAnyone at [https://localhost](https://localhost). 
Keep in mind that this URL points to localhost and will therefore only work if you try accessing it from within your server. 
Assuming that you're accessing your server via ssh, you should be able to access MaskAnyone via its IP (e.g. `https://<your-server-ip`) in your browser instead. 

## Setup (Worker Server)
*Only applicable for server setups, not local setups.* \
If you want to add additional worker servers to your setup, you can do so by following the steps below.

[tbd - not yet available]

## Interacting with the application after the initial setup
The setup steps outlined above are only necessary for the initial setup of the application.
Afterward you can interact with the application using the following commands.
- **Start the application**: `docker compose up -d`
- **Stop the application**: `docker compose down`
- **View the logs of a specific container**: `docker compose logs -f <container-name>`
- **View the status of all containers**: `docker compose ps`

> Remember to add ` -f docker-compose-local.yml` or ` -f docker-compose-server.yml` depending on your initial setup.


## Updating to a newer version of MaskAnyone
In general, if you want to update to newer version of the application, you have to do the following:
- Update the version in the `.env` file
- Check if the version update requires any additional changes
- Run `docker compose pull`
- Stop the application `docker compose down`
- Start the application again `docker compose up -d`

> Remember to add ` -f docker-compose-local.yml` or ` -f docker-compose-server.yml` depending on your initial setup.

