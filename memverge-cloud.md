# MM cloud instructions

Written by Ru Feng, Oct 2023


## **1. Download AWS CLI tools**
    (Linux/Windows/Mac) https://docs.aws.amazon.com/cli/latest/userguide/gteting-started-install.html
   - If you are **Mac** user, you can use below commands to install AWS CLI tools.
   ```
   curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
   sudo installer -pkg AWSCLIV2.pkg -target /
   ```

   - If you are **HPC/Linux** user, you can use to install AWS CLI tools (Also add `export PATH=$PATH:/home/<UNI>/.local/bin` in your `~/.bashrc` and don't forget to source it).
   ```
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   ./aws/install -i /home/<UNI>/.local/bin/aws-cli -b /home/<UNI>/.local/bin
   ```
   
   Check if it was installed successfully with 
   ```
   which aws
   aws --version
   ```






## **2. Create S3 Storage Resource and Upload Data to AWS**
### **FIXME:2.1 From memverge: this one-time job and don't need to do it every time (through GUI), there should be an admin to setup and manage it, after that, researchers and students only need to create a folder under that bucket**

### **2.1 Setting Up Your IAW User and Account (For admin)**
#### FIXME: for now I gave every one in the group the full access to the whole bucket, so everyone can read and edit others' file, that would be monvenient but also dangerous
- Log into AWS Console:
  - Navigate to [AWS Console](https://aws.amazon.com/).
  - Sign up for a root AWS account if you're new, else log in.

- Search for IAW:
  - After logging in, search for "IAW" using the top search bar.

- Creat Group
  - Click "User groups" on the left.
  - Attach "AmazonS3FullAccess" for this group
  - Add Users to this group.


- Add user and set up access key
  - GUI/or maybe for root user (first time to set up the access key) 

    - Add User:
      - Click "Users" on the left and then click "Create user" on the right.
      - Click "Next" following instructions.

    - Manage Access Keys:
      - Find "Security recommendations" on the IAW dashboard.
      - Click "Manage access keys".

    - Create an Access Key:
      - Go to the "Access keys" section.
      - Select "Create access key".

    - Retrieve Your Access Key and Secret Access Key:
      - A dialogue will show your Access Key ID and Secret Access Key.
      - Check the box, then click "Next".
      - Download a copy of these keys for safekeeping.
  - CLI (change to root access)
    ```
     aws iam create-user --user-name YourUserName
     aws iam add-user-to-group --user-name YourUserName --group-name Gao-lab
     aws iam create-access-key --user-name YourUserName
    ```
    copy these keys for safekeeping.
- Configure AWS CLI:
  - Run the following in your terminal:
    ```bash
    aws configure
    ```
  - Provide:
    - Your Access Key ID and Secret Access Key.
    - Region: `us-east-1`.
    - Output format (e.g., `yaml`).

### **2.2 Create an S3 Bucket (For admin)**

To create an S3 bucket, ensure your `$S3_BUCKET` name is globally unique and in lowercase. For example:
```bash
aws s3api create-bucket --bucket $S3_BUCKET --region $AWS_REGION
 # Example:
aws s3api create-bucket --bucket cumc-gao --region us-east-1

```


### **2.3 Copy Files from Local Directory to S3**
The files include your scripts(temporarly) and input data
Replace `$DATA_DIR`, `$S3_BUCKET` is the bucket we have set up, and `$UNI` as your personal folder. 
```bash
aws s3 cp $DATA_DIR s3://$S3_BUCKET/$UNI --recursive
 # Example:
aws s3 cp ~/codes/mmcloud/example s3://cumc-gao/test --recursive
```





## **3. Create Job Submission Script for each R Files**

The content of example script `run_job.sh` here:

 - **Prepare Data**: 
   This step is about copying data files from your S3 bucket to the worker node.
   
 - **Run the Test**:
    Execute your R script using the Micromamba command.
   
 - **Upload the Results**:
    After the job completion, copy results from the worker node back to your S3 bucket.

**Important**: 
- Within your `run_job.sh` script, make sure you update the `S3_BUCKET` variable to match the name of your S3 bucket:
  ```bash
  S3_BUCKET="s3://cumc-gao/test"
  ```


## **4. Set Up and Use Float Submit Commands**

Before submitting jobs, you'll need to set up and configure float for your environment. Here's how you can do that:
### **FIXME: add how to set up an op_center here, the step 1 in this [GUI tutorial](https://hackmd.io/@speri/rkmPkmP52#1-Launch-EC2-Instance) is nice but it would be better if you have a CLI instructions**
### **4.1 Download Float from the Operation Center:**

   ```bash
   wget https://<op_center_ip_address>/float --no-check-certificate
   # Example using an IP address:
   wget https://54.81.85.209/float --no-check-certificate

   ```

Or you can choose to open MMCloud OpCenter and download it with GUI.

### **4.2 Move and Grant Execution Rights to Float:**
  - For MAC user
   ```bash
   sudo mv float /usr/local/bin/
   sudo chmod +x /usr/local/bin/float
   alias float = /path/to/float_binary/float
   ```
  - For Linux user (Also add `export PATH=$PATH:<PATH>` in your `~/.bashrc` and don't forget to source it), there is a firewall issue on our HPC, so we can not login to float on HPC, the resolution for now is that we can submit data on through HPC, and submit commands/scripts locally (on your laptop or desktop)
  ```
  chmod +x <PATH>/float 
  ```
### **4.3 Addressing Mac Security Settings (Optional: For Mac Users):**

If you are using a Mac, float might be blocked due to your security settings. Follow these steps to address it:

   - Open 'System Preferences'.
   - Navigate to 'Privacy & Security '.
   - Under the 'Security' tab, you'll see a message about Float being blocked. Click on 'Allow Anyway'.

### **4.4 Rename Float to Avoid Name Conflicts (Optional For Mac Users, but would change your float name in downstream instruction and the future work):**

If there's an existing application or command named `float`, rename the downloaded `float` to avoid conflicts:

   ```bash
   mv /usr/local/bin/float /usr/local/bin/mmfloat
   alias mmfloat = /usr/local/bin/mmfloat
   ```

### **4.5 Log in to Float:**

There are two ways to login to Float: as admin or as a regular user. 

Use the username `admin` and password `memverge`:

   ```bash
   mmfloat login -a /<op_center_ip_address>
   # Example using an IP address:
   mmfloat login -u admin -p memverge -a 54.81.85.209
   ```
Create a new user for example `tom` and password `memverge`

   ```bash
    mmfloat user add tom
    New password:
    confirm password:
   ```
Login as the new user `tom`

   ```bash
   # Example login as Tom to the Opcenter with IP address:
   mmfloat login -u tom -p memverge -a 54.81.85.209
   ```

### **4.6 Create and Execute Float Submit Commands:**

This example command submits the `stephenslab_docker` container image with the `run_job.sh` job script. The job will have access to 2 CPUs, 8GB of Memory, and 10GB of EBS storage mounted on `/home/jovyan/aws/data`. Ensure you run this command in the directory where your data resides:

   ```bash
   mmfloat submit -i ghcr.io/cumc/stephenslab_docker -j run_job.sh -c 2 -m 8 --dataVolume "[size=10]:/home/jovyan/AWS/data"
   ```
`/home/jovyan/AWS/data` is the work dir of your job environment in your op_center



## **5. Submit Multiple Jobs Using Job Queue**

To efficiently manage and execute a high number of tasks, it's often useful to submit multiple jobs at once. Here's how you can set this up:

### **5.1 Create Multiple Job Scripts:**

Based on your requirements, you may have different scripts to run. For this example, let's assume you need multiple `run_job.sh` files with varying R script names.

- Repeat steps from sections 3 and 4 to create multiple `run_job.sh` files.
#### FIXME: this part is finished manually for now. 

- Generate a master script named run_multiple_jobs.sh that compiles all the mmfloat submit commands. The various run_job.sh scripts differ only in the specific R script they execute, which carries out the actual analysis :
    ```bash
    echo 'mmfloat submit -i ghcr.io/cumc/stephenslab_docker -j run_job.sh -c 2 -m 8 --dataVolume "[size=10]:/home/jovyan/AWS/data"' > run_multiple_jobs.sh
    echo 'mmfloat submit -i ghcr.io/cumc/stephenslab_docker -j run_job1.sh -c 2 -m 8 --dataVolume "[size=10]:/home/jovyan/AWS/data"' >> run_multiple_jobs.sh
    echo 'mmfloat submit -i ghcr.io/cumc/stephenslab_docker -j run_job2.sh -c 2 -m 8 --dataVolume "[size=10]:/home/jovyan/AWS/data"' >> run_multiple_jobs.sh
    ```

### **5.2 Use Job Manager to Submit Jobs in Bulk:**

Once you have your job scripts ready, you can use `jobmanager.sh` to submit them all.

- Ensure that you've edited the `jobmanager.sh` script to set `FLOAT="mmfloat"` and `OPCENTER=<op_center_ip_address>` (example:`OPCENTER="54.81.85.209"`).

- Give execution permissions to the scripts and then use `jobmanager.sh` to manage and submit the jobs:

    ```bash
    chmod +x run_multiple_jobs.sh ./jobmanager.sh
    ./jobmanager.sh -f run_multiple_jobs.sh 
    ```
- You can check your submitted jobs and job ID with 
  ```
  mmfloat squeue
  ```
- Also check the log file with 
  ```
  mmfloat log -j <jobID> ls #check log name 
  mmfloat log -j <jobID> cat <logfile> #cat log file
  ```
## **6. Download Job Results**

After submitting your jobs and once they are complete, you'll likely want to retrieve the results stored on S3 to your local machine. Here's how you can do that:

Ensure you've set or replaced the values for `$S3_BUCKET` and `$DATA_DIR` with their appropriate values. The basic structure of the command is:

```bash
aws s3 cp s3://$S3_BUCKET/$DATA_DIR/output $DATA_DIR/output --recursive
Example: 
aws s3 cp s3://cumc-gao/test/results ./aws-test/output --recursive
```

## **7. Clean up S3 buckets Example**

```
aws s3 rm s3://$S3_BUCKET/test/results --recursive
Example: 
aws s3 rm s3://cumc-gao/test/ --recursive
```




