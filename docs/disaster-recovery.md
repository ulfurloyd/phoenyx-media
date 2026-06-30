
## PVC recovery procedure

1. check btrfs snapshots
2. find old and new PVC directories
3. scale deployment to 0
4. move new dir to ./.bak
5. create replacement dir with new PVC's name
6. copy snapshot contents from old to new
7. scale deployment to 1
8. verify app

example:
```bash
#!/usr/bin/env bash
kubectl scale deploy soulsync -n media --replicas=0

sleep 0.1

sudo mv \
	/var/lib/rancher/k3s/storage/pvc-d39f9323-0b8d-43af-a659-2b52ca644679_media_soulsync-config \
	/var/lib/rancher/k3s/storage/pvc-d39f9323-0b8d-43af-a659-2b52ca644679_media_soulsync-config.bak

sleep 0.1

sudo mkdir \
	/var/lib/rancher/k3s/storage/pvc-d39f9323-0b8d-43af-a659-2b52ca644679_media_soulsync-config

sleep 0.1

sudo cp -a \
	/.snapshots/409/snapshot/var/lib/rancher/k3s/storage/pvc-b82c08ea-39b5-41ac-ad67-b771eafefa5e_media_soulsync-config/. \
	/var/lib/rancher/k3s/storage/pvc-d39f9323-0b8d-43af-a659-2b52ca644679_media_soulsync-config/

sleep 0.1

kubectl scale deploy soulsync -n media --replicas=1

sleep 0.1

kubectl rollout restart deployment/soulsync -n media
```
