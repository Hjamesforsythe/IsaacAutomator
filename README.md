![Isaac Automator](src/banner.png)

# Isaac Automator (v3)
# Hamilton Dev Version

Isaac Automator allows quick deployment of Isaac Sim and Isaac Lab to public clouds (AWS, GCP, Azure, and Alibaba Cloud are currently supported).

The result is a fully configured remote desktop cloud workstation that you can use to develop and test robotic applications within minutes and on a budget. Isaac Automator supports a variety of GPU instances and stop/start functionality to save on cloud costs and provides tools to aid your workflow (uploading and downloading data, autorun, deployment management, etc.).\

For this version we reccomend using the AWS pipeline, all other deployment method documentation has been removed but can be found in the Isaac and IsaacAutomator documentation

- [Installation](#installation)
  - [Installing Docker](#installing-docker)
    - [Windows WSL Requirement](#windows-wsl-requirement)
  - [Obtaining NGC API Key](#obtaining-ngc-api-key)
  - [Building the Container](#building-the-container)
    - [Linux/macOS](#linuxmacos)
    - [Windows](#windows)
- [Usage](#usage)
  - [Running Isaac Automator](#running-isaac-automator)
    - [Linux/macOS](#linuxmacos-1)
    - [Windows](#windows-1)
  - [Deploying Instances](#deploying-instances)
    - [AWS](#aws)
    - [GCP](#gcp)
    - [Azure](#azure)
    - [Alibaba Cloud](#alibaba-cloud)
  - [Connecting to Deployed Instances](#connecting-to-deployed-instances)
  - [Running Applications](#running-applications)
    - [Isaac Sim](#isaac-sim)
    - [Isaac Lab](#isaac-lab)
    - [Omniverse Isaac Gym Environments](#omniverse-isaac-gym-environments)
  - [Autorun Script](#autorun-script)
  - [Mapped Folders](#mapped-folders)
  - [Pausing and Resuming](#pausing-and-resuming)
  - [Uploading Data](#uploading-data)
  - [Downloading Data](#downloading-data)
  - [Repairing](#repairing)
  - [Destroying](#destroying)
- [Tips](#tips)
  - [Persisting Modifications to the Isaac Sim/Lab Environment](#persisting-modifications-to-the-isaac-simlab-environment)
  - [Updating Expired AWS Credentials](#updating-expired-aws-credentials)
  - [Storing API Keys Securely](#storing-api-keys-securely)

## Installation

### Installing Docker

Docker should be installed on your system. Visit <https://docs.docker.com/engine/install/> for installation instructions.

#### Windows WSL Requirement

On Windows, Docker Desktop requires **WSL 2 (Windows Subsystem for Linux)** to run Linux containers, which this project uses. To set up WSL:

1. **Install WSL 2**: Open PowerShell as Administrator and run:
   ```powershell
   wsl --install
   ```
   This installs WSL 2 with the default Ubuntu distribution. Restart your computer when prompted.

2. **Verify WSL 2 is installed**: After restarting, open a terminal and run:
   ```powershell
   wsl --version
   ```

3. **Configure Docker Desktop to use WSL 2**:
   - Open Docker Desktop
   - Go to **Settings** > **General**
   - Ensure **"Use the WSL 2 based engine"** is checked
   - Go to **Settings** > **Resources** > **WSL Integration**
   - Enable integration with your installed Linux distribution(s)
   - Click **Apply & Restart**

4. **Verify Docker is working with WSL**: Open a terminal and run:
   ```sh
   docker run --rm hello-world
   ```

> **Note**: Without WSL 2 properly configured, you may encounter errors when building or running the Isaac Automator container, as it relies on Linux-based containers (`--platform linux/x86_64`).

### Obtaining NGC API Key

An **NGC API Key** allows you to download Docker images from <https://ngc.nvidia.com/>. Obtain one at <https://ngc.nvidia.com/setup/api-key>.

### Building the Container

Please enter the following command in the project root directory to build the container:

#### Linux/macOS

```sh
./build
```

#### Windows

```sh
docker build --platform linux/x86_64 -t isaac_automator .
```

This will build the Isaac Automator container and tag it as `isaac_automator`.

## Usage

### Running Isaac Automator

#### Linux/macOS

On Linux and macOS there are two ways to run Isaac Automator commands:

1. First enter the Isaac Automator container and then run the command inside the container:

```sh
# enter Isaac Automator container
./run
# inside container:
./somecommand
```

2. Simply prepend the command with `./run`, like so:

```sh
./run ./somecommand <parameters>
```

for example:

```sh
./run ./deploy-aws
./run ./destroy my-deployment
```

#### Windows

On Windows, you can run Isaac Automator commands by entering the container first and then running the command inside the container, like so:

(enter Isaac Automator container)

```sh
docker run --platform linux/x86_64 -it --rm -v .:/app isaac_automator bash
```

(run the command inside the container)

```sh
./somecommand
```

### Deploying AWS Instances

#### AWS

<details>
  <a name="#aws-permissions"></a>
  <summary>Enabling Access Permissions</summary>

  You need _AmazonEC2FullAccess_ permissions enabled for your AWS user. You can enable those in the [Identity and Access Management (IAM) section](https://console.aws.amazon.com/iamv2/home#/home) of the AWS console, as follows:

  1. Go to <https://console.aws.amazon.com/iamv2/home#/home>
  2. Click "Access Management" \> "Users" in the left menu
  3. Search for your user name
  4. Under "Permissions" tab click "Add permissions"
  5. Choose "Attach existing policies directly"
  6. Search for _AmazonEC2FullAccess_, check the box next to it, click "Next"
  7. Click "Add permissions"
</details>

<details>
  <a name="#aws-access-creds"></a>
  <summary>Getting Access Credentials</summary>
  You will need an _AWS Access Key_ and _AWS Secret Key_ for an existing account. You can obtain those in the [Identity and Access Management (IAM) section](https://console.aws.amazon.com/iamv2/home#/home) of the AWS console.
</details>

If you have completed the above steps or already have your permissions and credentials set up, run the following command in the project root directory:

```sh
# enter Isaac Automator container
./run
# inside container:
./deploy-aws
```

Tip: Run `./deploy-aws --help` to see more options.

##### Using Temporary Credentials

If you are using temporary credentials that may expire and prevent you from deleting the deployment or stopping/starting the instance (e.g., from `aws sts assume-role`), you can manually edit the `/app/state/<deployment-name>/.tfvars` file in the Automator container, like so:

```sh
# inside container:
nano /app/state/<deployment_name>/.tfvars
```

Then set the `aws_access_key_id`, `aws_secret_key`, and `aws_session_token` variables to the new values.

### Connecting to Deployed Instances

Deployed Isaac Sim instances can be accessed via:

- SSH
- noVNC (browser-based VNC client)
- NoMachine (remote desktop client)

Look for the connection instructions at the end of the deployment command output. Additionally, this information is saved in the `state/<deployment-name>/info.txt` file.

You can view available arguments with the `--help` switch for the start scripts. In most cases, you won't need to change the defaults.

Tip: You can use the `./connect <deployment-name>` helper command to connect to the deployed instance via SSH.

### Running Applications

To use the installed applications, connect to the deployed instance using noVNC or NoMachine. You can find the connection instructions at the end of the deployment command output. Additionally, this information is saved in the `state/<deployment-name>/info.txt` file.

#### Isaac Sim

Isaac Sim will automatically start when the cloud VM is deployed. Alternatively, click the "Isaac Sim" icon on the desktop, or run the following command in a terminal on the deployed instance:

```sh
~/isaacsim.sh
```

To get a shell inside the Isaac Sim container, click the "Isaac Sim Shell" icon on the desktop. Alternatively, run the following command in a terminal on the deployed instance:

```sh
~/isaacsim-shell.sh
```

#### Isaac Lab

[Isaac Lab](https://isaac-sim.github.io/IsaacLab/) can be pre-installed on deployed instances. To install a specific version of Isaac Lab, provide a valid Git reference from <https://isaac-sim.github.io/IsaacLab/> as the value of the `--lab` parameter to the deployment command.

To run Isaac Lab, click the "Isaac Lab" icon on the desktop or run the following command in the terminal:

```sh
~/isaaclab.sh
```

#### Omniverse Isaac Gym Environments

_Omniverse Isaac Gym Environments is deprecated in favor of Isaac Lab._

[Omniverse Isaac Gym Reinforcement Learning Environments for Isaac Sim](https://github.com/NVIDIA-Omniverse/OmniIsaacGymEnvs) ("Omni Isaac Gym Envs") can be pre-installed on deployed instances.

To run Omniverse Isaac Gym Environments, click the "Omni Isaac Gym Envs" icon on the desktop or run the following command in the terminal:

```sh
~/omni-isaac-gym-envs.sh
```

The default output directory (`/OmniIsaacGymEnvs/omniisaacgymenvs/runs`) in the OmniIsaacGymEnvs container will be linked to the default results directory (`/home/ubuntu/results`) on the deployed instance. You can download the contents of this directory to your local machine using the `./download <deployment_name>` command.

Tip: To install a specific version of OmniIsaacGymEnvs, provide a valid reference from <https://github.com/NVIDIA-Omniverse/OmniIsaacGymEnvs> as the value of the `--oige` parameter to the deployment command. For example, to install the `devel` branch on an AWS instance, run the following command:

```sh
./deploy-aws --oige devel
```

### Autorun Script

By default, Isaac Sim will start when the cloud VM is deployed.

If you want to launch a custom application or script on startup, modify the [`uploads/autorun.sh`](uploads/autorun.sh) script (on your local machine). It will either be uploaded to the cloud VM automatically, or you can upload it manually using the `./upload` command.

Every time the cloud VM is deployed or started from a stopped state, the `autorun.sh` script will be executed.

This functionality can be useful for running batch jobs, generating data on startup, or preparing the environment for the user.

### Mapped Folders

The following folders are mapped to the running Isaac Sim container by default (container paths may differ for specific applications):

- `/home/ubuntu/uploads` (host) --> `/uploads` (container) - user data uploaded to the deployment with the `./upload` command or automatically from the local `uploads/` folder
- `/home/ubuntu/results` (host) --> `/results` (container) - results of applications run on the deployment; you can download them from the deployed machine with the `./download` command
- `/home/ubuntu/workspace` (host) --> `/workspace` (container) - workspace folder; can be used to exchange data between the host and the container

### Pausing and Resuming

You can stop and restart instances to save on cloud costs. To do so, run the following commands:

```sh
# enter Isaac Automator container
./run
# inside container:
./stop <deployment-name>
./start <deployment-name>
```

### Uploading Data

You can upload user data from the `uploads/` folder (in the project root) to the deployment by running the following command:

```sh
# enter Isaac Automator container
./run
# inside container:
./upload <deployment-name>
```

Data will be uploaded to the `/home/ubuntu/uploads` directory by default, on all deployed instances. You can change this by passing the `--remote-dir` argument to the command. Run `./upload --help` to see more options.

### Downloading Data

You can download user data to the `results/` folder (in the project root) from deployed instances by running the following command:

```sh
# enter Isaac Automator container
./run
# inside container:
./download <deployment-name>
```

Data will be downloaded from the `/home/ubuntu/results` directory by default. You can change this by passing the `--remote-dir` argument to the command. Run `./download --help` to see more options.

### Repairing

If, for some reason, the deployment cloud resources or software configuration become corrupted, you can attempt to repair the deployment by running the following commands:

```sh
# run both terraform and ansible
./repair <deployment-name>
# just run terraform to try fixing the cloud resources
./repair <deployment-name> --no-ansible
# just run ansible to try fixing the software configuration
./repair <deployment-name> --no-terraform
```

### Destroying

To destroy a deployment, run the following command:

```sh
# enter Isaac Automator container
./run
# inside container:
./destroy <deployment-name>
```

You will be prompted to enter the deployment name to destroy.

_Please note that information about the deployed cloud resources is stored in the `state/` directory. Do not delete this directory._

## Tips

### Persisting Modifications to the Isaac Sim/Lab Environment

It's common to require that modifications to the Isaac Lab and Isaac Sim source code, as well as installed custom components, persist across instance shutdowns, restarts, and redeployments.

To achieve that, you can do the following:

Go to the `/uploads/` directory in the repo and find the `autorun.sh` file.

Modify its contents like so (this example customizes the Isaac Lab environment):

```sh
#!/bin/sh

# This script is executed when
# 1. the VM is first deployed
# 2. the VM is started after being stopped

SELF_DIR="$(dirname $0)"

docker image tag isaaclab:latest isaaclab:stock
docker build -t isaaclab:latest -f "${SELF_DIR}/isaaclab-custom.dockerfile" "${SELF_DIR}"
```

Now, create `isaaclab-custom.dockerfile` in the same `uploads/` directory with your changes and the first line being:

```dockerfile
FROM isaaclab:stock
```

Every time you deploy or start an instance (using `./start`), the `uploads/` directory will be uploaded and `autorun.sh` executed, which will build the customized Isaac Lab environment.

Alternatively, you can push your custom Docker image to a registry and pull it from the autorun script, tagging it `isaaclab:latest`, like so:

```sh
docker pull your-custom-image
docker tag your-custom-image isaaclab:latest
```

### Updating Expired AWS Credentials

If you are using temporary AWS credentials that expire (e.g., from `aws sts assume-role`), you may be unable to stop, start, or destroy the deployed instance after the credentials expire. To fix this, you can manually edit the `state/<deployment-name>/.tfvars` file in the Automator container, updating the `aws_access_key_id`, `aws_secret_key`, and `aws_session_token` variables to the new values.

You can do this by running the following command:

```sh
# inside container:
nano /app/state/<deployment_name>/.tfvars
```

### Storing API Keys Securely

When working with AWS credentials and NGC API keys, follow these best practices to keep your keys secure:

**General Guidelines:**
- **Never commit keys to version control** - Add credential files to `.gitignore` and avoid hardcoding keys in scripts
- **Use environment variables** - Store keys in environment variables rather than plaintext files when possible
- **Limit key permissions** - Use IAM roles with minimum required permissions (e.g., only _AmazonEC2FullAccess_ for this project)
- **Rotate keys regularly** - Periodically generate new keys and revoke old ones, especially if you suspect exposure

**Platform-Specific Storage:**

| Platform | Recommended Storage Location |
|----------|------------------------------|
| Windows  | Use Windows Credential Manager or store in `%USERPROFILE%\.aws\credentials` for AWS keys |
| macOS    | Use Keychain Access or store in `~/.aws/credentials` for AWS keys |
| Linux    | Use a secret manager (e.g., `pass`, `secret-tool`) or store in `~/.aws/credentials` with `chmod 600` permissions |

**AWS Credentials File Example** (`~/.aws/credentials`):

```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

**NGC API Key**: Store your NGC API key in an environment variable:

```sh
# Linux/macOS - add to ~/.bashrc or ~/.zshrc
export NGC_API_KEY="your-ngc-api-key"

# Windows - set via System Properties > Environment Variables
# Or in PowerShell:
[Environment]::SetEnvironmentVariable("NGC_API_KEY", "your-ngc-api-key", "User")
```

> **Future Development**: Planned security enhancements include integrated secrets management and support for cloud-native key vaults (AWS Secrets Manager, Azure Key Vault). Contributions welcome!
