# Disable containerd Image Store and Switch to overlay2 on Ubuntu

If Docker is not recognizing your cert/key after placing them in the correct folder, you may need to disable the containerd image store and switch back to `overlay2`.

---

## 1️⃣ Edit Docker Daemon Configuration

Open or create the Docker daemon configuration file:

```
sudo nano /etc/docker/daemon.json
```

Add (or merge) the following configuration:

```
{"features": {"containerd-snapshotter": false} } 
```

> ⚠️ If the file already contains JSON, make sure you properly merge this into the existing structure (do not overwrite other settings).

Save and exit.

---

## 2️⃣ Restart Docker

Restart the Docker service to apply the changes:

```
sudo systemctl restart docker
```

---

## 3️⃣ Verify Storage Driver

Run the following command:

```
docker info | grep -i "storage driver" 
```

You should see:

```
Storage Driver: overlay2 
```

And you should **NOT** see:

```
driver-type: io.containerd.snapshotter.v1
```

If `overlay2` is shown and the containerd snapshotter reference is gone, the configuration change was successful.