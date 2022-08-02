web-service.yml
----------
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose -f docker-compose.yml convert
    kompose.version: 1.26.1 (a9d05d509)
  creationTimestamp: null
  labels:
    io.kompose.service: web
  name: web
spec:
  ports:
    - name: "3000"
      port: 3000
      targetPort: 3000
  selector:
    io.kompose.service: web
status:
  loadBalancer: {}
---------------

web-deployment.yml
---------
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose -f docker-compose.yml convert
    kompose.version: 1.26.1 (a9d05d509)
  creationTimestamp: null
  labels:
    io.kompose.service: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: web
  strategy: {}
  template:
    metadata:
      annotations:
        kompose.cmd: kompose -f docker-compose.yml convert
        kompose.version: 1.26.1 (a9d05d509)
      creationTimestamp: null
      labels:
        io.kompose.network/network-backend: "true"
        io.kompose.service: web
    spec:
      containers:
        - image: web
          name: web
          ports:
            - containerPort: 3000
          resources: {}
      restartPolicy: Always
status: {}
-------------------

network-backend-networkpolicy.yaml
-------------------------

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: null
  name: network-backend
spec:
  ingress:
    - from:
        - podSelector:
            matchLabels:
              io.kompose.network/network-backend: "true"
  podSelector:
    matchLabels:
      io.kompose.network/network-backend: "true"
---------------------

mongodb-data-persistentvolumeclaim.yaml
------------------------
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: mongodb-data
  name: mongodb-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
status: {}
--------------------

mongo-deployment.yaml
---------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose -f docker-compose.yml convert
    kompose.version: 1.26.1 (a9d05d509)
  creationTimestamp: null
  labels:
    io.kompose.service: mongo
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: mongo
  strategy:
    type: Recreate
  template:
    metadata:
      annotations:
        kompose.cmd: kompose -f docker-compose.yml convert
        kompose.version: 1.26.1 (a9d05d509)
      creationTimestamp: null
      labels:
        io.kompose.network/network-backend: "true"
        io.kompose.service: mongo
    spec:
      containers:
        - env:
            - name: MONGODB_INITDB_ROOT_PASSWORD
              value: password
            - name: MONGODB_INITDB_ROOT_USERNAME
              value: username
          image: mongo
          name: mongo
          resources: {}
          volumeMounts:
            - mountPath: /data/db
              name: mongodb-data
      restartPolicy: Always
      volumes:
        - name: mongodb-data
          persistentVolumeClaim:
            claimName: mongodb-data
status: {}
-------------------


