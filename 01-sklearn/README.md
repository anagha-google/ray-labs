# Lab: Distributed sklearn on Ray

Ray parallelizes sklearn and you can find documetation from the Ray community [here](https://docs.ray.io/en/latest/ray-more-libs/joblib.html#ray-joblib).

## 1. About the lab

### 1.1. Use Case

![M1-1](./images/skl-01.png)   
<br><br>

<hr>

### 1.2. What is showcased?

![M1-1](./images/skl-02.png)   
<br><br>

<hr><hr>

## 2. Setting up the lab environment

Disclaimer: At the time of creation of this lab (Feb-Mar 2024), the cluster provisioned via the UI had ray 2.4, while the SDK offered ray 2.9, and the Colab runtime template created by the Ray cluster had ray 2.9. Adjustments have been made in the lab for ray 2.4 to be consistent as the version across colab and the cluster.<br>

### 2.1. What's involved?

![M1-1](./images/skl-03.png)   
<br><br>

<hr>


### 2.2. What gets provisioned?


![M1-1](./images/skl-04.png)   
<br><br>

<hr>


### 2.3. Instructions for provisioning & smoke testing

Follow each step in the sequence laid out in the [instructions](https://github.com/anagha-google/ray-labs/blob/main/00-common/Module-00-Provisioning.md)

<hr>

### 2.4. Time taken

~30 minutes

<hr><hr>

## 3. Lab Modules

### 3.1. Baseline model training without ray

Duration: 90 minutes or < | [Lab guide](module-01-baseline-sans-ray-README.md)

![M1-1](./images/skl-3.1a.png)   
<br><br>

![M1-1](./images/skl-3.1b.png)   
<br><br>

![M1-1](./images/skl-3.1c.png)   
<br><br>

<hr><hr>

### 3.2. ray.data Google Cloud Storage primer

Duration: 5 minutes or < | [Lab guide](module-02-ray-data-gcs-primer-README.md)

![M1-1](./images/skl-3.2a.png)   
<br><br>

![M1-1](./images/skl-3.2b.png)   
<br><br>



<hr><hr>


### 3.3. ray.data Google Cloud Storage primer

Duration: 5 minutes or < | [Lab guide](module-03-ray-data-bq-primer-README.md)

![M1-1](./images/skl-3.3a.png)   
<br><br>

![M1-1](./images/skl-3.3b.png)   
<br><br>



<hr><hr>
