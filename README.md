
### DOCKER IMAGE  ###
docker build -t taylanbakircioglu/jmeter-base:latest -f dockerfiles/Dockerfile-base .
docker build -t taylanbakircioglu/jmeter-master:latest -f dockerfiles/Dockerfile-master .
docker build -t taylanbakircioglu/jmeter-slave:latest -f dockerfiles/Dockerfile-slave .

### DEPLOY KUBERNETES  ###
kubectl create ns test-jmeter
kubectl apply -f k8s-yamls/sa.yaml -n test-jmeter
kubectl apply -f k8s-yamls/configmap-master.yaml -n test-jmeter

kubectl apply -f k8s-yamls/deploy-master.yaml -n test-jmeter
kubectl apply -f k8s-yamls/deploy-slave.yaml -n test-jmeter
kubectl apply -f k8s-yamls/service-slave.yaml -n test-jmeter

kubectl get pods -n test-jmeter



### DELETE ###
kubectl delete -f k8s-yamls/sa.yaml -n test-jmeter
kubectl delete -f k8s-yamls/configmap-master.yaml -n test-jmeter

kubectl delete -f k8s-yamls/deploy-master.yaml -n test-jmeter
kubectl delete -f k8s-yamls/deploy-slave.yaml -n test-jmeter
kubectl delete -f k8s-yamls/service-slave.yaml -n test-jmeter


### RUN LOAD TEST ###
kubectl cp scenario jmeter-master-6b54645dc6-vbgxc:/jmeter/apache-jmeter-5.5/bin -n test-jmeter
kubectl exec -it deploy/jmeter-master -n test-jmeter -- bash
cp /script/loadtest.sh $JMETER_HOME/bin/
cd $JMETER_HOME/bin/
./loadtest.sh scenario/sample.jmx