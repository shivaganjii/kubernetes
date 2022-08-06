# Install rook storage;

	$ cd rook/cluster/examples/kubernetes/api

	$ nano operator.yaml

		line326: ROOK_ENABLE_DISCOVERY_DAEMON -> true

# Create crds, common and operator file;

	$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml

Check pods is ready:

	$ kubectl -n rook-api get pod   ## verify the rook-api-operator is in the `Running` state before proceeding to cluster.yaml

# Create cluster file;

	$ kubectl create -f cluster.yaml
	$ kubectl -n rook-api get pod

***** image list pods *****

# Create toolbox file;

	$ kubectl create -f toolbox.yaml

	$ kubectl -n rook-api rollout status deploy/rook-api-tools

	$ kubectl -n rook-api exec -it deploy/rook-api-tools -- api status

	$ kubectl -n rook-api exec -it deploy/rook-api-tools -- api osd status

	$ kubectl -n rook-api exec -it deploy/rook-api-tools -- api df

	$ kubectl -n rook-api exec -it deploy/rook-api-tools -- rados df

	$ kubectl create -f csi/rbd/storageclass.yaml

# Test and check storageclass;

	$ kubectl create -f ../mysql.yaml

	$ k get sc

	$ k get pvc

	$ k get pv

	$ k get svc

# Delete test pods;
	$ kubectl delete -f ../wordpress.yaml

	$ kubectl delete -f ../mysql.yaml

# Create filesystem file;

	$ kubectl create -f filesystem.yaml  ## To confirm the filesystem is configured, wait for the mds pods to start

# Test and check;

	$kubectl -n rook-api get pod -l app=rook-api-mds

	$ kubectl -n rook-api exec -it deploy/rook-api-tools -- api status

# Create storageclass file;

	$ kubectl create -f csi/apifs/storageclass.yaml

# Test and check finaly;

	$ kubectl -n rook-api get service

	$ kubectl -n rook-api get secret rook-api-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo

	$ kubectl create -f dashboard-external-https.yaml

	$ kubectl -n rook-api get service
