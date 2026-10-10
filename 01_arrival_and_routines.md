# BIOKT Lab Handbook, Chapter 1: Arrival, Accounts and Group Routines

**Status:** v1 · **Last revised:** 2026-10-10

---

## 1. Why this chapter exists

Your first weeks in a group are full of things nobody thinks to tell you,
because everyone else stopped noticing them years ago: which accounts you need,
who to ask for them, when the group meets and what you are expected to bring.

This chapter covers two things:

- **The administrative minimum** (§2–3): what to set up when you arrive so that
  you can start working. This part is deliberately short. The paperwork itself
  belongs to the hiring institution, and its staff are the authority on it.
- **How the group runs** (§4–7): the routines we have agreed on, and why they
  work the way they do. This is the part worth reading carefully.

---

## 2. Your first weeks: the administrative minimum

Work through this list in your first week or two. Some items take days to come
through, so start them early, and tell the PI if one gets stuck.

> **To be completed.** Institution-specific steps (who to contact, which forms,
> links) are still to be written. Until then, ask the PI.

- [ ] **Contract and HR paperwork** with the hiring institution (UPV/EHU or
      DIPC). Keep a copy of everything you sign.
- [ ] **Institutional ID and email.** Use the institutional address for anything
      work-related, including cluster and software accounts.
- [ ] **Slack.** Ask for an invitation to the group workspace (§6).
- [ ] **Group machines.** Ask for an account on the group workstations and GPU
      servers, and set up SSH access (§3).
- [ ] **HPC clusters.** If your project needs them, request accounts on the
      UPV/EHU and DIPC clusters (§3.4). Start early: both need approval
      or a signature from someone else.
- [ ] **GitHub.** Ask to be added to the group organisation, where project
      repositories live (Chapter 2, §6).
- [ ] **Group meetings and one-to-ones.** Make sure you are in the calendar
      invitation for the weekly group meeting and have agreed a slot for your
      one-to-one with the PI (§4–5).
- [ ] **Read Chapter 2** (project organisation) before you create your first
      project.

---

## 3. Remote access with SSH

Almost all computational work in the group happens on remote machines, reached
over SSH from your own computer. Set this up once, properly, and you will not
think about it again.

> **Note.** Host names, user-name conventions and any VPN or gateway
> requirements will be given to you on arrival rather than written here.

### 3.1 Getting an SSH client

| Your system | What to use |
|---|---|
| **macOS** | Built in. Open Terminal and use `ssh`. |
| **Linux** | Built in (install `openssh-client` if it is missing). |
| **Windows** | Three options, described below: the **Ubuntu terminal** (WSL), **Git Bash**, or the OpenSSH client built into **PowerShell**. |

On Windows, all three options give you `ssh`, `scp`, `ssh-keygen` and a
`~/.ssh/config` file, so any of them is enough to log in to the group machines.
The differences show up once you do more than log in:

| | Ubuntu terminal (WSL) | Git Bash | PowerShell (OpenSSH) |
|---|---|---|---|
| What it is | A full Ubuntu Linux running inside Windows (Windows Subsystem for Linux) | A small Unix-like shell installed with Git for Windows | Windows' own shell, with the OpenSSH client built in |
| How to get it | `wsl --install` from an administrator PowerShell, then open "Ubuntu" from the Start menu | Install [Git for Windows](https://gitforwindows.org) | Already there on Windows 10/11 |
| `ssh`, `scp`, `ssh-keygen` | yes | yes | yes |
| `ssh-copy-id` | yes | yes | no |
| `rsync` | yes | no | no |
| `git` | yes (`sudo apt install git`) | yes | only if Git for Windows is installed |
| Bash scripts, `grep`/`sed`/`awk` | yes | yes, but only the basic tools | no (PowerShell has a different syntax) |
| Installing more software | `apt`, `conda`, `pip`: the same as on the group machines | not really | not really |
| Where your SSH keys live | in the Linux home directory, separate from Windows | `C:\Users\<you>\.ssh`, shared with PowerShell | `C:\Users\<you>\.ssh`, shared with Git Bash |

**Our recommendation:** use the **Ubuntu terminal**. It is the only option that
behaves exactly like the group machines, so commands, scripts and `rsync`
transfers from this handbook work unchanged. **Git Bash** is a reasonable
lightweight alternative if you only need SSH and Git. Use plain PowerShell only
to log in.

Two practical notes on WSL:

- Keep your working files inside the Linux file system (`~/...`), not under
  `/mnt/c/...`. Accessing Windows files from Linux works, but it is much slower.
- WSL keeps its own `~/.ssh`. If you have already created keys in Git Bash or
  PowerShell, either generate a new pair in WSL or copy them across, and fix
  the permissions afterwards (`chmod 600 ~/.ssh/id_ed25519`).

### 3.2 Keys, not passwords

Generate a key pair once per computer you work from:

```bash
ssh-keygen -t ed25519 -C "your.name@institution"
```

Protect the private key with a passphrase. The private key (`~/.ssh/id_ed25519`)
never leaves your computer; the public key (`~/.ssh/id_ed25519.pub`) is what you
copy to each remote machine:

```bash
ssh-copy-id user@host            # macOS, Linux, WSL, Git Bash
```

From PowerShell, where `ssh-copy-id` is not available, append the
contents of your `.pub` file to `~/.ssh/authorized_keys` on the remote machine.

### 3.3 A config file

Typing full host names gets old quickly. Put an entry per machine in
`~/.ssh/config`:

```
Host short-name
    HostName full.host.name
    User your-user-name
    IdentityFile ~/.ssh/id_ed25519
```

after which `ssh short-name`, `scp file short-name:` and `rsync` all just work.

**Why bother:** keys are more secure than passwords, they let scripts and
`rsync` run without prompting you, and a config file means everyone in the group
refers to machines by the same short names, which makes instructions and
scripts portable.

### 3.4 The institutional HPC clusters

Beyond the group's own machines, we run on two institutional services. Each
has its own account procedure and documentation, which are the authority:
what follows is only the minimum to get you started, and it may go out of date.
If the two disagree, trust their documentation, and please tell us so we can fix
this section.

**UPV/EHU: Scientific Computing Service (ARINA)**
([documentation](https://scc.ehu.eus/))

- **Getting an account:** fill in the online form at
  [ehu.eus/sgi/CUENTA](https://www.ehu.eus/sgi/CUENTA). Students use the
  "new account request with guarantee" option, which the PI submits for you,
  so ask the PI to start it. Requests are reviewed, not granted automatically.
  Details: [Accounts](https://scc.ehu.eus/access/account/).
- **Before connecting:** you must be on the UPV/EHU network, either from a
  computer on campus or through the university VPN.
  Details: [Connection](https://scc.ehu.eus/access/connect/).

**DIPC: Supercomputing Center**
([documentation](https://scc.dipc.org/docs/getting-started/quick-start/))

- **Getting an account:** download the
  [account request form](https://scc.dipc.org/docs/getting-started/files/dipc_cluster_account_form.pdf),
  complete sections 1 and 2 and sign it, then give it to your institution's HR
  representative (at the university, the department secretary). They confirm
  your affiliation and contract end date, sign, and forward it to DIPC. Accounts
  are usually created within a few working days of DIPC receiving the form.
  Details: [Accounts and access](https://scc.dipc.org/docs/getting-started/accounts/).
- **Before connecting:** register your public SSH key on their authentication
  server, as described in
  [Account setup](https://scc.dipc.org/docs/getting-started/account-setup/).
  Key login is required.
- **Renewal:** the account expires on the end date stated on the form. If your
  contract is extended, submit a new form.

On both services, you agree to their usage rules when you get an account.
Read their best-practice pages before your first production run.

---

## 4. The weekly group meeting

The whole group meets once a week. Each meeting has two parts:

1. **A short update from every member**, about five minutes each: what you did
   this week, what worked, what didn't, what you are doing next.
2. **One longer presentation** by a single member on the status of their
   project. The long slot follows a rota, so everyone knows well in advance when
   their turn is. It is sometimes replaced by a journal club.

**What the group meeting is for:** visibility. Everyone should have a rough idea
of what everyone else is doing, so that you know who to ask when you hit a
problem someone else has already solved, and so that the group spots overlaps
and connections between projects.

**What it is not for:** detailed supervision. If your update turns into a
twenty-minute debugging session, that conversation belongs in your one-to-one
(§5) or on Slack (§6).

Some practical advice for the short update:

- **Show, don't describe.** One figure says more than five minutes of talking.
  Your lab notebook (§7) should already contain it.
- **Report what didn't work.** A failed simulation or a dead end is useful
  information for everyone, and saying it out loud is often how someone else
  recognises a problem they have seen before.
- **Say what is blocking you.** The meeting is a good place to find out that
  someone has the script, the parameter file or the reference you need.

Before showing a figure, run through the "Before you show a figure in group
meeting" checklist in Chapter 2, §9.

---

## 5. One-to-one meetings

Every member has a regular one-to-one meeting with the PI:

| Position | Frequency |
|---|---|
| PhD and MSc students | weekly |
| Postdocs | every two weeks |

**This is where supervision actually happens.** Scientific questions, problems
with your project, career advice, and anything you would rather not raise in
front of the whole group.

### How a one-to-one runs

- **Nothing is prepared in writing beforehand.** You do not write a report for
  the meeting. Bring your lab notebook (§7) and talk through it.
- **The last five minutes produce a to-do list**: what you will do before the
  next meeting. The PI keeps these lists as a running, dated record, newest
  first.
- **The next meeting opens by reading that list back**: what got done, what
  didn't, and why.

**Why no pre-meeting report:** a report written for a meeting turns the lab
notebook into a deliverable, and a notebook written for someone else stops being
a place where you think honestly. The notebook is yours (Chapter 2, §7.2). The
to-do list closes the loop instead: it is short, it is agreed together, and
reading it back makes it obvious when a task keeps slipping, which is usually a
sign that something needs to change, not that someone is failing.

---

## 6. Slack

Slack is the group's always-on channel between meetings. Use it for:

- quick questions to the PI or to anyone else in the group;
- sharing a figure or a result you want an opinion on without waiting for the
  next meeting;
- practical coordination: machines, queues, who is using which GPU.

Ask early. A question that takes someone else two minutes to answer can cost
you two days of searching on your own. Nobody in the group thinks less of a
person who asks; everyone has wasted time by not asking.

---

## 7. The lab notebook

The recommended way to keep track of your work is a lab notebook: figures,
mistakes, references and thoughts, **written as the work happens**. It is
described in detail in Chapter 2, §7.2. In short:

- writing is how you think, and how you find out whether you understand
  something;
- it makes progress visible as it happens, rather than only in an eventual paper;
- it captures what you would otherwise forget;
- it is what you bring to your one-to-one (§5) and what you show in group
  meeting (§4).

The format is your choice (a Markdown file, a Word document, a Google Doc). It
is **not a monitoring tool**, and nobody grades it.

---

## 8. Self-check

### End of your first week

- [ ] Your contract and HR paperwork are under way, and you know who to ask
      about them.
- [ ] You are on Slack and in the group meeting invitation.
- [ ] You have a weekly or fortnightly one-to-one slot agreed with the PI.
- [ ] You can `ssh short-name` into every group machine you need, with a key.
- [ ] You know when your first long slot in group meeting is.
- [ ] You have started a lab notebook, even if it has only one entry.

### End of your first month

- [ ] You have accounts on any HPC cluster your project needs.
- [ ] You are in the group GitHub organisation, and your first project follows
      Chapter 2.
- [ ] You have brought your notebook to at least one one-to-one and left with a
      to-do list.
