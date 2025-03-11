+++
title = 'Tweak Dev - Setup'
date = 2025-01-24
weight = 2
summary = "How to setup the tools we need"

[cover]
image = "cover.png"
hiddenInList = true
+++

To avoid this tutorial becoming outdated I wont show step-by-step how to install the tools we'll be using
but rather direct you to each tool's own installation steps

## Theos

Firstly lets setup the build system we'll be using

Installation is as easy as installing some packages and running a install script
The guide for installing can be found at [theos.dev/docs/installation](https://theos.dev/docs/installation)

After installing you'll need to ensure the `$THEOS` environment variable is configured in your shell

## Mac VM

If you're on a mac you can [skip this section](#ssh)

If you're on linux you'll need to setup a Mac <abbr title="Virtual Machine">VM</abbr> to avoid the
[abi issues](https://blog.rugmj.dev/posts/tweak-dev/overview#new-abi-pains) I had mentioned in the last post

I use [kholia/OSX-KVM](https://github.com/kholia/OSX-KVM) for the actual VM itself, you can find the installation steps on their github page

### Samba

To share my linux file system to the Mac VM I'm hosting a samba server,
you can install it from your distro's package manager and copy my configuration:

```conf
# /etc/samba/smb.conf

[global]
    workgroup = RUGMJ
    fruit:copyfile = yes
    security = user
    map to guest = bad user
    dns proxy = no
    server role = standalone server
    inherit permissions = yes
    vfs objects = fruit catia streams_xattr acl_xattr
    map acl inherit = yes
    fruit:aapl = yes
    fruit:model = MacSamba
    fruit:metadata = stream
    readdir_attr:aapl_rsize = yes
    readdir_attr:aapl_finder_info = yes
    readdir_attr:aapl_max_access = yes

[root]
    path = /
    valid users = rugmj
    read only = no
    guest ok = no

    force directory mode = 0775
    force create mode = 0664

    # Fruit config
    fruit:posix_rename = yes
    fruit:veto_appledouble = yes
    fruit:nfs_aces = yes
    fruit:wipe_intentionally_left_blank_rfork = no
```

On the mac side of things I have a cron entry which looks like this:

```crontab
@reboot sleep 60;/sbin/mount_smbfs //rugmj:mypassword@192.168.1.233/root /Users/rugmj/linux-root
```

Where `rugmj` is my host user's username

`mypassword` is the password I use on my host

`192.168.1.233` is the local ip address of my host

`/Users/rugmj/linux-root` is the path, on the mac, where I want my linux file system mounted

### Shell command

In order to make running commands on the mac easier I have setup a simple shell command which lets me run a command over ssh

It's used my calling `mac the_command_you_want_to_run with -its --args`

#### Bash / ZSH

```bash
mac () {
  ssh -p 2222 127.0.0.1 "source ~/.zshrc && cd /Users/rugmj/linux-root$(pwd) && $@"
}
```

#### Fish

```fish
function mac
  ssh -p 2222 127.0.0.1 "source ~/.zshrc; cd /Users/rugmj/linux-root(pwd); $argv"
end
```

#### Nu

```nu
def mac [...args] {
    let args = $args | str join ' '
    ssh -p 2222 127.0.0.1 $"source ~/.zshrc && cd /Users/rugmj/linux-root(pwd) && ($args)"
```

### Setting up theos, on the mac vm

Although we've setup theos on our host we'll also need to set it up on the mac itself

The mac specific installation steps can be found at [theos.dev/docs/installation-macos](https://theos.dev/docs/installation-macos)

#### Symlinks

There's some paths in the `$THEOS` directory which we'd want to keep synced across our host and mac installations of theos

To do this we'll make use of symlinks

On the mac VM, run:

```bash
rm -rf ~/theos/lib
ln -s ~/linux-root/opt/theos/lib ~/theos/lib

rm -rf ~/theos/include
ln -s ~/linux-root/opt/theos/include ~/theos/include

rm -rf ~/theos/sdks
ln -s ~/linux-root/opt/theos/sdks ~/theos/sdks
```

Changing `~/linux-root/opt/theos` to the path to your host's theos installation
and `~/theos` to your mac's theos installation

## Ssh

In order to install our tweaks onto our device easily we'll want to setup ssh between our host, mac, and jailbroken device

#### Mac VM

You can enable ssh in the settings app on the VM.

My VM's network configuration will route port `22` on the mac to port `2222` on my host.
So to connect to the VM over ssh I run `ssh -p rugmj@127.0.0.1`.
If your networking configuration is different you'll have to alter any ssh-related commands in the next steps

#### Jailbroken Device

Install the `openssh-server` package from your package manager and enable any ssh-related options in your jailbreak

Theos has an environment variable you can set `$THEOS_DEVICE_IP` which will let it know where to ssh into in order to install your package

You should add this on your host's and on your mac VM's shell config

#### Ssh keys

To make the process of installing packages as seamless as possible I've setup ssh keys between all 3 (or 2 if you're not using a VM) devices

To create a key (skip this if you already have an ssh key setup) you can run `ssh-keygen`, when asked for a password leave it blank.
Although it's not the most secure it'll avoid the annoying repetition of your password,
if you'd rather keep a password I'd recommend setting up an `ssh-agent`

To copy your key to your devices, on your host, run:

```bash
ssh-copy-id -p 2222 rugmj@127.0.0.1 # Mac VM

ssh-copy-id mobile@$THEOS_DEVICE_IP # user 'mobile' on your jailbroken device
ssh-copy-id root@$THEOS_DEVICE_IP # user 'root'
```

Lets test it,

Running `ssh mobile@THEOS_DEVICE_IP` should work without asking for a password

## Sourcekit-lsp

We chose swift, mainly, for it's LSP support so how do we set it up?

Well that depends on your editor

#### Neovim

I use neovim so it was as easy as setting the relevant options in `lspconfig`

```lua
require 'lspconfig'.sourcekit.setup {
  on_attach = on_attach,
  cmd_env = { SOURCEKIT_TOOLCHAIN_PATH = vim.fn.expand("$THEOS/toolchain/linux/host") },
  cmd = { vim.fn.expand("$THEOS/toolchain/linux/host/bin/sourcekit-lsp") }
}
```

#### VSCode

> [!TODO]
> Write vscode sourcekit setup instructions

## Flex

Flex is the <abbr title="Reverse Engineering">RE</abbr> tool we'll be using the most, it's very easy to setup.

You can either install [VolumeFlex](https://havoc.app/package/volumeflex), which you activate by pressing both volume buttons at the same time.

Or you can install [FLEXing](https://github.com/NSExceptional/FLEXing), which you either hold the status bar or tap with 3 fingers to activate.

Up to you which you use, I use volumeflex.

## Conclusion

Now that we've got everything setup you're ready to write your first tweak.
In the next post I'll give you a step by step guide on how to do it.
