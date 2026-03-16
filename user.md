# Managing Sudo Users in Ubuntu

This guide provides step-by-step instructions to create a new user, grant them **sudo (administrator) privileges**, and completely remove them from an Ubuntu system.

---

# Part 1: Creating a New User with Sudo Privileges

In Ubuntu, it is recommended to use the `adduser` command rather than `useradd`.  
`adduser` is an interactive script that automatically creates the user's home directory and prompts you to set a password.

## 1. Create the New User

Replace `newuser` with your desired username.

You will be prompted to enter and confirm a password, followed by optional user information.  
You can press **Enter** to skip the optional fields.

```bash
sudo adduser newuser
```

---

## 2. Grant the User Sudo Privileges

In Ubuntu, any user added to the `sudo` group automatically gains administrator privileges.

Use the `usermod` command to append (`-a`) the user to the `sudo` group (`-G`).

```bash
sudo usermod -aG sudo newuser
```

---

## 3. Verify the New User's Sudo Access

Switch to the new user account.

```bash
su - newuser
```

Run a command that requires root privileges, such as listing the contents of the root directory.

You will be asked to enter the password you just created.

```bash
sudo ls -la /root
```

If the command executes successfully, the user has sudo privileges.

Return to the original administrator account.

```bash
exit
```

---

# Part 2: Completely Removing the User

When removing a user, it is best practice to delete the **home directory** and **mail spool** to ensure no orphaned files remain on the system.

---

## 1. Ensure the User Is Not Logged In

A user cannot be deleted if they are currently logged in or running processes.

If necessary, terminate all processes running under that user.

```bash
sudo killall -u newuser
```

---

## 2. Delete the User and Their Home Directory

Use the `deluser` command with the `--remove-home` flag.

This removes:
- the user account
- the user's primary group
- the `/home/newuser` directory

```bash
sudo deluser --remove-home newuser
```

---

## 3. (Optional) Search for Remaining Files

The `--remove-home` flag removes most user data. However, files created outside the home directory may still exist and will belong to the deleted user's UID.

You can locate these files using:

```bash
sudo find / -uid $(id -u newuser) -print
```

**Note:**  
If you run this command after deleting the user, you must use the numeric UID that belonged to the user because the system will no longer recognize the username.
