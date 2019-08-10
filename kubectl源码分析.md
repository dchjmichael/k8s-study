入口文件：
cmd/kubectl/kubectl.go

# root命令初始化
这里的root命令就是`kubectl`命令本身，底层依赖的是cobra包创建命令。

创建命令的链路：

NewDefaultKubectlCommand --> NewDefaultKubectlCommandWithArgs --> NewKubectlCommand

在`NewKubectlCommand`里首先是用cobra创建一个基本的`kubectl`命令：

```go
cmds := &cobra.Command{
		Use:   "kubectl",
		Short: i18n.T("kubectl controls the Kubernetes cluster manager"),
		Long: templates.LongDesc(`
      kubectl controls the Kubernetes cluster manager.

      Find more information at:
            https://kubernetes.io/docs/reference/kubectl/overview/`),
		Run: runHelp,
		// Hook before and after Run initialize and write profiles to disk,
		// respectively.
		PersistentPreRunE: func(*cobra.Command, []string) error {
			return initProfiling()
		},
		PersistentPostRunE: func(*cobra.Command, []string) error {
			return flushProfiling()
		},
		BashCompletionFunction: bashCompletionFunc,
	}
```

我们知道`kubectl`还包含了很多子命令，此时的cmds还没有包含任何其他命令，它是通过命令组的方式进行添加的：

```go
groups := templates.CommandGroups{
		{
            //基本命令
			Message: "Basic Commands (Beginner):",
			Commands: []*cobra.Command{
				create.NewCmdCreate(f, ioStreams),
				expose.NewCmdExposeService(f, ioStreams),
				run.NewCmdRun(f, ioStreams),
				set.NewCmdSet(f, ioStreams),
			},
		},
		{
            //基本命令
			Message: "Basic Commands (Intermediate):",
			Commands: []*cobra.Command{
				explain.NewCmdExplain("kubectl", f, ioStreams),
				get.NewCmdGet("kubectl", f, ioStreams),
				edit.NewCmdEdit(f, ioStreams),
				delete.NewCmdDelete(f, ioStreams),
			},
		},
		{
            //部署命令
			Message: "Deploy Commands:",
			Commands: []*cobra.Command{
				rollout.NewCmdRollout(f, ioStreams),
				rollingupdate.NewCmdRollingUpdate(f, ioStreams),
				scale.NewCmdScale(f, ioStreams),
				autoscale.NewCmdAutoscale(f, ioStreams),
			},
		},
		{
            //集群管理命令
			Message: "Cluster Management Commands:",
			Commands: []*cobra.Command{
				certificates.NewCmdCertificate(f, ioStreams),
				clusterinfo.NewCmdClusterInfo(f, ioStreams),
				top.NewCmdTop(f, ioStreams),
				drain.NewCmdCordon(f, ioStreams),
				drain.NewCmdUncordon(f, ioStreams),
				drain.NewCmdDrain(f, ioStreams),
				taint.NewCmdTaint(f, ioStreams),
			},
		},
		{
            //排错调试命令
			Message: "Troubleshooting and Debugging Commands:",
			Commands: []*cobra.Command{
				describe.NewCmdDescribe("kubectl", f, ioStreams),
				logs.NewCmdLogs(f, ioStreams),
				attach.NewCmdAttach(f, ioStreams),
				cmdexec.NewCmdExec(f, ioStreams),
				portforward.NewCmdPortForward(f, ioStreams),
				proxy.NewCmdProxy(f, ioStreams),
				cp.NewCmdCp(f, ioStreams),
				auth.NewCmdAuth(f, ioStreams),
			},
		},
		{
            //高级命令
			Message: "Advanced Commands:",
			Commands: []*cobra.Command{
				diff.NewCmdDiff(f, ioStreams),
				apply.NewCmdApply("kubectl", f, ioStreams),
				patch.NewCmdPatch(f, ioStreams),
				replace.NewCmdReplace(f, ioStreams),
				wait.NewCmdWait(f, ioStreams),
				convert.NewCmdConvert(f, ioStreams),
				kustomize.NewCmdKustomize(ioStreams),
			},
		},
		{
            //设置命令
			Message: "Settings Commands:",
			Commands: []*cobra.Command{
				label.NewCmdLabel(f, ioStreams),
				annotate.NewCmdAnnotate("kubectl", f, ioStreams),
				completion.NewCmdCompletion(ioStreams.Out, ""),
			},
		},
    }
    //把所有组中的命令添加到root中
    groups.Add(cmds)
```

除了这些命令，还有些其他命令：

```go
cmds.AddCommand(alpha)
cmds.AddCommand(cmdconfig.NewCmdConfig(f, clientcmd.NewDefaultPathOptions(), ioStreams))
cmds.AddCommand(plugin.NewCmdPlugin(f, ioStreams))
cmds.AddCommand(version.NewCmdVersion(f, ioStreams))
cmds.AddCommand(apiresources.NewCmdAPIVersions(f, ioStreams))
cmds.AddCommand(apiresources.NewCmdAPIResources(f, ioStreams))
cmds.AddCommand(options.NewCmdOptions(ioStreams.Out))
```