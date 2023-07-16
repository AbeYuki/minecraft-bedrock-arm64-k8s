# minecraft-bedrock-arm64-k8s
Run the minecraft bedrock server in an ARM64 environment.

## Quick start
```
cd minecraft-bedrock-arm64-k8s/overlay/dev/
```

```
cat <<'EOF'> secret/allowlist.json 
[]
EOF
```

```
cat <<'EOF'> secret/permissions.json 
[]
EOF
```

```
cat <<'EOF'> secret/server.properties
server-name=Dedicated Server
gamemode=survival
force-gamemode=false
difficulty=easy
allow-cheats=false
max-players=10
online-mode=true
allow-list=false
server-port=19132
server-portv6=19133
view-distance=32
tick-distance=4
player-idle-timeout=30
max-threads=8
level-name=Bedrock level
level-seed=
default-player-permission-level=member
texturepack-required=false
content-log-file-enabled=false
compression-threshold=1
server-authoritative-movement=server-auth
player-movement-score-threshold=20
player-movement-action-direction-threshold=0.85
player-movement-distance-threshold=0.3
player-movement-duration-threshold-in-ms=500
correct-player-movement=false
server-authoritative-block-breaking=false
EOF
```
```
vi kustomization.yaml
```
Modify Storage, storageClassName
```
patchesStrategicMerge:
- |-
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: frontend-app01
  spec:
    accessModes:
    - ReadWriteMany
    resources:
      requests:
        storage: <MODIFY>
    storageClassName: <MODIFY>
    volumeMode: Filesystem
```

```
kubectl apply -f namespace.yaml
```

```
kubectl apply -k ./
```


# backup

backup with longhorn is not possible.  
Restoring from backup fails to mount.  

```
NS=prod-minecraft
POD_NAME=$(kubectl get pod -n $NS --template='{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
```

To local
```
kubectl cp $NS/$POD_NAME:/bedrock-server/worlds ./bedrock_world_data_$(date +%Y%m%d)
```

To container
```
kubectl cp bedrock_world_data/ $NS/$POD_NAME:/bedrock-server/worlds
```

Login pod  
```
kubectl exec -it -n $NS pod/$POD_NAME -- bash
```

Restore
```
root@prod-minecraft-frontend-app01-001-b984468cd-q2qnc:/bedrock-server# cd worlds/
root@prod-minecraft-frontend-app01-001-b984468cd-q2qnc:/bedrock-server/worlds# ls
BedrockServer  bedrock_world_data_20230716  lost+found
root@prod-minecraft-frontend-app01-001-b984468cd-q2qnc:/bedrock-server/worlds# cp -ar bedrock_world_data_20230716/BedrockServer/ ./
```
