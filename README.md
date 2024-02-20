# MaskAnyone - Production Setup
This document describes the process of setting up a production environment for MaskAnyone.

## Prerequisites
Make sure you have installed [Docker](https://docs.docker.com/get-docker/) on your system and set the appropriate permissions. 
Furthermore, please also ensure that you have sufficient hard drive space available for the setup.
We recommend at least 50GB of free space.

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

**Step 4: Pull/build the infrastructure.**
Run the following command to pull all the images and prepare the application infrastructure:
```bash
docker-compose pull && docker-compose build
```

**Step 5: Start the application for the first time.**
To do so first start the database container.
```bash
docker-compose up -d postgres
```
Then wait for 10 seconds to give it some time to prepare the database. 
Afterward, also start the other containers, except for the workers.
```bash
docker-compose up -d proxy frontend backend keycloak
```
Then wait for another 30 seconds to give keycloak some time to complete its initial setup.
Now try to access the keycloak admin UI at [https://localhost/auth/](https://localhost/auth/). 
Remember that localhost will only work if the browser runs on the same computer where you're doing the setup.
If you're doing the setup on a server via SSH, then you can try accessing it using `https://<your-server-ip>/auth/`.

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

**Step 8: Start the remaining containers.**
```bash
docker-compose up -d
```

**Step 9: Verify that MaskAnyone is running.**
First check that all containers are up and running:
```bash
docker-compose ps
```
If you find that a container is not running, you can try to identify the issue by looking at its logs:
```bash
docker-compose logs -f <container-name>
```
If your containers are running, then please try accessing MaskAnyone at [https://localhost](https://localhost). 
Keep in mind that this URL points to localhost and will therefore only work if you try accessing it from within your server. 
Assuming that you're accessing your server via ssh, you should be able to access MaskAnyone via its IP (e.g. `https://<your-server-ip`) in your browser instead. 

## Setup (Worker Server)
*Only applicable for server setups, not local setups.* \
If you want to add additional worker servers to your setup, you can do so by following the steps below.

[tbd]

## Interacting with the application after the initial setup
The setup steps outlined above are only necessary for the initial setup of the application.
Afterward you can interact with the application using the following commands.
- **Start the application**: `docker-compose up -d`
- **Stop the application**: `docker-compose down`
- **View the logs of a specific container**: `docker-compose logs -f <container-name>`
- **View the status of all containers**: `docker-compose ps`

## Updating to a newer version of MaskAnyone
In general, if you want to update to newer version of the application, you have to do the following:
- Update the version in the `.env` file
- Check if the version update requires any additional changes
- Run `docker-compose pull && docker-compose build`
- Stop the application `docker-compose down`
- Start the application again `docker-compose up -d`
