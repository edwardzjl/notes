# Resolve too many open files

Sometimes when I use `kubectl logs -f` I notice a `too many open files` error and the log stream was immediately closed after the error message.

Recently I also experienced a pod creation error like [this](https://github.com/kubeflow/manifests/issues/2087).

According to the error message, I think I need to increase open file limit on my worker nodes.
Following [this comment](https://github.com/kubeflow/manifests/issues/2087#issuecomment-1004285387)
and [this guide](https://www.ibm.com/docs/en/ahte/4.0?topic=wf-configuring-linux-many-watch-folders)
I first checked that `max_user_watches` on my worker nodes are `8192`:

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

and the `max_user_instances` are `128`:
```bash
cat /proc/sys/fs/inotify/max_user_instances
```

both were too small I think.

So I increased `max_user_watches` to `524288` (which is both in the example, and my wsl's default):

```bash
sudo sh -c 'echo "fs.inotify.max_user_watches=524288" >> /etc/sysctl.conf'
```

and `max_user_instances` to `1024`:

```bash
sudo sh -c 'echo "fs.inotify.max_user_instances=1024" >> /etc/sysctl.conf'
```

Finally reload config:

```bash
sudo sysctl -p /etc/sysctl.conf
```

After reloading, the pod start running successfully.

I also recommend using `ansible` to automate these operations, as you need to operate on all worker nodes. But it's out of this note's scope.
