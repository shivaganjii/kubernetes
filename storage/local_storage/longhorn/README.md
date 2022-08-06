![image](https://user-images.githubusercontent.com/82608620/183246873-0ef3b9a3-0ac3-4ecc-b0ae-aa739df47bea.png)


# Install Longhorn by helm;
1- add repository:

	$ helm repo add longhorn https://charts.longhorn.io 

2- Update repository:

	$ helm repo update

3- If you want change default config:

	$ cd ./long/longhorn/

and change "values.yaml"

4- Create namespace:

	$ kubectl create ns longhorn-system

5- Install Long horn by Helm:

	$ helm install longhorn longhorn/longhorn -n longhorn-system -f values.yaml

6- Check components:

	$ kubectl get pod -n longhorn-system -wo wide 

	$ kubectl get svc -n longhorn-system  

	$ kubectl get storageclass 
