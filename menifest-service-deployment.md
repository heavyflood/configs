# SERVICE
apiVersion: v1
kind: Service
metadata:
  name: ${APP_NAME}-service
  namespace: ${NAMESPACE}
spec:
  type: NodePort
  selector:                         
    app: ${APP_NAME}                
  ports:
  - name: http
    protocol: TCP
    port: 18080                      
    targetPort: 18080                
                                    
---
# DEPOLOYMENT
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
spec:
  replicas: 9
  revisionHistoryLimit: 1           
  strategy:
    type: RollingUpdate             
    rollingUpdate:                  
      maxSurge: 1                   
      maxUnavailable: 0
  selector:                         
    matchLabels:
      app: ${APP_NAME}
  template:                         
    metadata:
      labels:
        app: ${APP_NAME}
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: ncloud.com/nks-nodepool
                  operator: In
                  values:
                    - ${NODE_NAME}
      containers:
      - name: ${APP_NAME}
        image: ${IMAGE}
        env:
        - name: NODE_IP
          valueFrom: {fieldRef: {fieldPath: status.hostIP}}
        - name: NODE_NAME
          valueFrom: {fieldRef: {fieldPath: spec.nodeName}}
        - name: POD_NAME
          valueFrom: {fieldRef: {fieldPath: metadata.name}}
        - name: OKIND
          value: ${APP_NAME}
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 18080
        resources:
          requests:           
            memory: "512Mi"                                                
            cpu: "500m"
          limits:             
            memory: "1024Mi"                                                
            cpu: "1000m"
        livenessProbe:              
          exec:
            command: ["sh", "-c", "cd /"]              
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          exec:
            command: ["sh", "-c", "cd /"]  
          initialDelaySeconds: 30
          periodSeconds: 15      
        lifecycle:               
          preStop:
            exec:
              command: ["sh", "-c", "sleep 20"]             
      imagePullSecrets:
      - name: regcred-${NAMESPACE}

---
# HPA(Horizonal Pod AutoScaler)
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: ${APP_NAME}-hpa
  namespace: ${NAMESPACE}           
spec:                               
  maxReplicas: 18
  minReplicas: 9
  scaleTargetRef:                   
    apiVersion: apps/v1          
    kind: Deployment                  
    name: ${APP_NAME}
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 70
