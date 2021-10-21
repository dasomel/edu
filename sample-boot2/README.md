## 1. /k8s/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: sample-boot2
      labels:
        app: sample-boot2
    spec:
      replicas: 2
      revisionHistoryLimit: 1
      selector:
        matchLabels:
          app: sample-boot2
      template:
        metadata:
          labels:
            app: sample-boot2
        spec:
          containers:
            - name: sample-boot2
              image: dasomel/sample-boot2:version
              ports:
                - containerPort: 8080
              imagePullPolicy: Always
              resources:
                requests:
                  cpu: 0.5
                  memory: 0.5Gi
                limits:
                  cpu: 0.5
                  memory: 0.5Gi
## 2. /k8s/service.yml
    apiVersion: v1
    kind: Service
    metadata:
      name: sample-boot2
    spec:
      selector:
        app: sample-boot2
      ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
## 3. build & dockerizing
    ./gradlew clean war bootRepackage
    docker build . -t dasomel/sample-boot2:2.0.23
    docker push dasomel/sample-boot2:2.0.23
## 4. kubernetes build & deploy
    kubectl apply -f ./k8s/deployment.yaml
    kubectl apply -f ./k8s/service.yaml

## 1. local-ingress.yaml
##### kubectl apply -f ./k8s/local-ingress.yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: opdc-ingress
    namespace: default
    annotations:
        kubernetes.io/ingress.class: nginx
    spec:
      rules:
        - host: opdc.io
          http:
            paths:
              - path: /sample
                backend:
                  serviceName: egov-sample
                  servicePort: 8080
              - path: /maven
                backend:
                  serviceName: sample-boot
                  servicePort: 8080
        - host: opdc2.io
          http:
            paths:
              - path: /gradle
                backend:
                  serviceName: sample-boot2
                  servicePort: 8080
## 2. Confirm
    http://opdc2.io/gradle/egovSampleList.do