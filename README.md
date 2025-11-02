# openshift-ai-model-training-steps

- Create data science project e.g. **ai-ops**

![image-20251101194328800](assets/image-20251101194328800.png)

- Create OpenShift access token

  ```
  oc -n ai-ops create sa automation
  
  oc -n ai-ops adm policy add-cluster-role-to-user cluster-admin -z automation
  
  echo "
  apiVersion: v1
  kind: Secret
  metadata:
    name: secret-sa-automation
    annotations:
      kubernetes.io/service-account.name: "automation" 
  type: kubernetes.io/service-account-token 
  " | oc -n ai-ops apply -f -
  
  oc -n ai-ops get secrets secret-sa-automation -o jsonpath='{.data.token}' | base64 -d
  ```

  **Keep** this token for future use

  ```
  eyJhbGciOiJSUzI1NiIsImtpZCI6Il93SXVmSGhkcm04bWUyTVEwOUtYbXY4UGppcUV4dVRpNXVrUlBHN2NyOTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3Viab.....
  
  ...
  
  cCh4V_00
  ```

- Create bucket from minio and get **Access Key**

  Create  `models` bucket

  ![image-20251101200009597](assets/image-20251101200009597.png)

  Create access key

  ![image-20251101200219128](assets/image-20251101200219128.png)

  keep this access key for future use

  ![image-20251101200307755](assets/image-20251101200307755.png)

- Configure pipeline server

  create `pipeline` bucket

  ![image-20251101212310924](assets/image-20251101212310924.png)

  configure server with s3 access

  ![image-20251101212806726](assets/image-20251101212806726.png)

  ![image-20251101212818241](assets/image-20251101212818241.png)

- Create notebook 

  named e.g. **notebook**

![image-20251101194812977](assets/image-20251101194812977.png)

Request GPU **if you want to speed up the model testing from the notebook**

![image-20251101203937113](assets/image-20251101203937113.png)

Create RWX storage with Storage Class **efs-sc**

![image-20251101194958365](assets/image-20251101194958365.png)

mount as **shared**

![image-20251101195024065](assets/image-20251101195024065.png)

![image-20251101195201672](assets/image-20251101195201672.png)

add **Environment variables**

- `HF_TOKEN`
- `OPENSHIFT_API_TOKEN`
- `MODEL_REGISTRY_URL`

![image-20251101205726707](assets/image-20251101205726707.png)

add s3 connection

![image-20251101200723384](assets/image-20251101200723384.png)

![image-20251101200933108](assets/image-20251101200933108.png)

notebook is created

![image-20251101201055612](assets/image-20251101201055612.png)

- setup pipeline

access notebook and clone pipeline code from `https://github.com/JonkeyGuan/openshift-ai-model-training-pipeline.git`

![image-20251101213454177](assets/image-20251101213454177.png)

switch to **openshift-ai-model-training-pipeline** folder

![image-20251101215143170](assets/image-20251101215143170.png)

you can try to run each step notebook directly from JupyterLab or view detail of each step, but we will run them from pipeline.

double click your pipeline file to edit it

![image-20251101215213989](assets/image-20251101215213989.png)

change **Environment Variables** of each **NODE PROPERTIES** then save

- `HF_TOKEN`
- `OPENSHIFT_API_TOKEN`
- `MODEL_REGISTRY_URL`
- `AWS_S3_ENDPOINT`
- `AWS_DEFAULT_REGION`
- `AWS_S3_BUCKET`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

![image-20251101214804181](assets/image-20251101214804181.png)

run pipeline

![image-20251101215315360](assets/image-20251101215315360.png)

if you meet below that means the pipeline server is configured after the workbench, you can stop the running workbench, make a minor edit (e.g., add/delete a dummy environment variable), save the changes, and then restart the workbench.

![image-20251101222409215](assets/image-20251101222409215.png)

the normal step will be 

![image-20251101222546454](assets/image-20251101222546454.png)

adjust pipeline parameters and click ok

![image-20251101222653859](assets/image-20251101222653859.png)

![image-20251101222735978](assets/image-20251101222735978.png)

if faced below ssl error, you can modify pipeline runtime cloud object storage endpoint manually 

![image-20251101223453218](assets/image-20251101223453218.png)

change `https` to `http` or change to actual https://minio.your-domain with public certs will be more better.

![image-20251101223704472](assets/image-20251101223704472.png)

run pipeline again

![image-20251101223849744](assets/image-20251101223849744.png)

view **Run Details**

![image-20251101224207236](assets/image-20251101224207236.png)

Finished pipeline run

![image-20251101231804905](assets/image-20251101231804905.png)

registered new tuned model 

![image-20251101231845198](assets/image-20251101231845198.png)

new model is running

![image-20251101232039422](assets/image-20251101232039422.png)

underlying pod workload of the model

![image-20251101232005087](assets/image-20251101232005087.png)

get model id and verify its status 

![image-20251101232347664](assets/image-20251101232347664.png)

- visusalize training with tensorboard

  > if tensor board is not setup yet
  >
  > ```
  > oc -n <your data science project namespace> apply -f https://raw.githubusercontent.com/JonkeyGuan/openshift-ai-model-training-infra/refs/heads/main/tensorboard/tensorboard.yaml
  > ```

  access tensorboard web url e.g. `https://tensorboard-ai-ops.apps.<cluster-name>.<your-domain>``

  ![image-20251102111938396](assets/image-20251102111938396.png)

  



