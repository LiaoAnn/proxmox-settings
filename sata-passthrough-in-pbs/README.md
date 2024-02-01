# Sata Passthrough in PBS

> Translated with Copilot.

I didn't passthrough the Sata Controller to PBS when I was setting up PBS, instead, I chose to make it thin-lvm and then give it to PBS directly. There is no disadvantage for me, but it is a bit annoying, so I opened this note to record how to do it.

## Create PBS VM

Since I have to redo it anyway, let's just reinstall it.
This part is almost done by creating a VM, let's deal with the hard drive part first.

## Passthrough Sata Drive

### Step 1: Check all the hard drives

```bash
lsblk
```

And then check to see if the hard drive has a partition, if so, deal with it.

### Step 2: Find out the Model and Serial of the hard drive

```bash
lsblk -o +MODEL,SERIAL
```

After that, you will see two columns added to the table, `MODEL` and `SERIAL`, record these two pieces of information for the hard drive you want.

### Step 3: Find out the specific path of the hard drive

```bash
ls /dev/disk/by-id
```

If it is a general Sata drive, its name will start with `ata`, such as `ata-{model}_{serial}`.

### Step 4: Setting Passthrough

Finally, we have come to this part. Here we use `qm set` to add the hard drive directly to the VM.

```bash
qm set <vm_id> -<name> <path>
```

Like this:

```bash
qm set 101 -scsi1 /dev/disk/by-id/ata-CT1000MX500SSD1_2328E6EE55CF
```

After adding it, you can see a `scsi1` hard drive added to the **Hardware** page. If you want to add more later, the name should be different, like `scsi2` or something.

## Install & Setup PBS

### Step 1: update repositories

I found that I can use GUI to do this, so I used GUI to do this.

Go to Administration > Repositories, then click Add, add `No Supscription`, then disable enterprise disabled below, then go to the side of Updates and click Refresh, and finally click Upgrade.
This has the same effect as `apt update && apt dist-upgrade -y`, don't forget to reboot after updating.

### Step 2: Init Drive

At first, PBS didn't know where to store the backup data. You can see that the Datastore will display `No Datastores configured`. At this time, you need to add Datastore.

Go to Storage / Disks to add a hard drive. Note that you need to remember to check **Add as Datastore** (although it is checked by default).

After adding, you can go to Datastore to see that there is a basic dashboard to see.

### Step 3: Add PBS to PVE

Go to PVE GUI first, select Datacenter > Storage, click Add above and select Proxmox Backup Server.

The ID is usually its name, you can see what you want to take.
Server is its IP Addr, like `192.168.100.100`.
Username is usually `root@pam`.
Password is the user's password.
Datastore is the name of the Datastore just now.
The Fingerprint part needs to go to the PBS GUI, click Dashboard > Show Fingerprint, copy and paste it to PVE.

After filling in, click Add, and you will see that the PBS just added to the left Sidebar.
At this time, you can go to the PBS GUI, click Dashboard > Show Fingerprint, and you will see that there is a PVE added to the list.

### Step 4: Add Schedule

PBS can back up once in a fixed cycle. The setting method is PVE GUI > Datacenter > Backup, click Add above.

Schedule is to set when to back up, there are many options to play with.
And then you can choose which VM/CT to backup below.

> If you are using PVE, **do not add PBS to the list** when you check it, it will not run the backup, and even the whole PBS will die, don't ask me how I know.

## Refs

- [How to run TrueNAS on Proxmox? - YouTube](https://www.youtube.com/watch?v=M3pKprTdNqQ&ab_channel=ChristianLempa)
- [How to setup PBS to backup your Proxmox VMs and Containers (youtube.com)](https://www.youtube.com/watch?v=Px5eHcUKbbQ&ab_channel=ElectronicsWizardry)
