#!/usr/bin/env groovy
@Library(["Common@generic-pipeline-v2.1"])
// @Library(["Common@dev"])
// @Library(["Common@rsync-x"])
// @Library(["Common@github-support"])

import common.vNext.*
import common.vNext.tools.*
import common.vNext.enums.*
import common.vNext.helpers.*
import common.vNext.settings.*
import common.vNext.configurations.*
import groovy.transform.Field


//will be excluded from upload, but will be deleted if present on the remote if DeleteExcluded is true
ArrayList<String> excludedItems = [
    'docs/',
    '.env',
    '.env.sample'
]

//protected files and folders that will never be deleted
ArrayList<String> protectedItems = [
    'docs/',
    'docs/**',
    '.env'
]

String mainBranchName = "master",
    devBranchName = "dev",
    abWireguardBranchName = "ab"
    cacheParameterName = "UseCache"


PipelineSettingsX pipelineSettings = new PipelineSettingsX(NodeX.Debian1Slave)
pipelineSettings.GitRemoteCredentialId = CredentialsX.BitbucketCloudBasic
// PipelineSettingsX pipelineSettings = new PipelineSettingsX("debian-x")
pipelineSettings.SlackLiveChannel = '#ci-general'
pipelineSettings.SlackDevChannel = '#ci-general'
pipelineSettings.IsLiveSimulation = 'true' //!!! change back to false
pipelineSettings.ChatClientType = ChatClientTypeX.Mattermost

pipelineSettings
    .AddDeploymentTool(DeploymentToolX.Rsync)
    .WithStepName("Deploy")
    .WithExecutionTimeout(TimeSpan.FromMinutes(20))
    .WithOption("--no-o --no-g --no-t")

pipelineSettings
    .WithMainBranchName(mainBranchName)
    .WithDevBranchName(devBranchName)
    //use dry-run only when confirmation is required or when you want to determine if deploy is required at all
    .WithDryRunFor(mainBranchName, devBranchName, abWireguardBranchName)
    .IncludeInBuild("PR~", mainBranchName, devBranchName, abWireguardBranchName)
    // .WithVersionLocator(TextLocatorX.Create('version.php', /(?<=semVer = ")[^"]*(?=")/))
    // .WithVersionChangeFor(mainBranchName)
    .WithReleaseNotificationEmails('')
    .WithFailureNotificationEmails('')
    .When({ -> env.BRANCH_NAME == devBranchName || env.BRANCH_NAME == abWireguardBranchName },
    { ps -> ps.WithParameter(
        booleanParam(name: cacheParameterName, defaultValue: true, description: 'Use cache when building the image'),
        { paramSettings -> paramSettings.WithName(cacheParameterName) })
    })
    .UseConfigurationFactory({ PipelineSettingsX ps ->
        // protectedItems.add(ps.VersionLocator.FileName())
        def configuration = new ConfigurationX()

        if (ps.IsDevBranch()) {
            if (ps.IsMainBuildId()) {

                Boolean cacheValue =  params.get(cacheParameterName)
                String dockerBuildOptions = cacheValue ? "" : "--no-cache"
                String url = "https://wireguard.perspecta.org/"
                ps.SetEnvironment(EnvironmentX.Development)
                    .WithBaseUrl(url)
                    .AddConfiguration(
                        ConfigurationX.Create()
                        .WithUrl(url)
                        // .AddProperty("rSyncSelector", "v2")
                        .WithDeployment(
                            DeploymentX.Create()
                                .WithHost(HostX.Create('192.168.56.1', 22))
                                .WithUsername("jenkins")
                                .WithCredentialId("global-linux")
                                .WithBaseDestinationPath("/home/jenkins")
                                .WithDestinationPath("/wg-manager")
                                .WithSourceFilesRelativePath(ps.SourceFilesRelativePath)
                                .WithExcludedItems(excludedItems)
                                .WithProtectedItems(protectedItems)
                                .DeleteExcluded(),
                            { deployment ->
                                deployment
                                    .RemoteExecuteAfterDeploy("cd ${deployment.BaseDestinationPath}${deployment.DestinationPath}")
                                    .RemoteExecuteAfterDeploy("docker build ${dockerBuildOptions} -t ghristov88/wg-manager -t ghristov88/wg-manager:latest-${currentBuild.number} -f Dockerfile .")
                                    .RemoteExecuteAfterDeploy("echo ''")
                                    .RemoteExecuteAfterDeploy("time docker-compose -f docker-compose.yml up -d")
                            })
                    )
            }
        }
        else if (ps.IsCurrentBranch(abWireguardBranchName)) {
            if (ps.IsMainBuildId()) {

                Boolean cacheValue =  params.get(cacheParameterName)
                String dockerBuildOptions = cacheValue ? "" : "--no-cache"
                ps.SetEnvironment(EnvironmentX.Development)
                    .AddConfiguration(
                        ConfigurationX.Create()
                        // .AddProperty("rSyncSelector", "v2")
                        .WithDeployment(
                            DeploymentX.Create()
                                .WithHost(HostX.Create('157.245.28.100', 22))
                                .WithUsername("jenkins")
                                .WithCredentialId("global-linux")
                                .WithBaseDestinationPath("/home/jenkins")
                                .WithDestinationPath("/wg-manager")
                                .WithSourceFilesRelativePath(ps.SourceFilesRelativePath)
                                .WithExcludedItems(excludedItems)
                                .WithProtectedItems(protectedItems)
                                .DeleteExcluded(),
                            { deployment ->
                                deployment
                                    .RemoteExecuteAfterDeploy("cd ${deployment.BaseDestinationPath}${deployment.DestinationPath}")
                                    .RemoteExecuteAfterDeploy("docker build ${dockerBuildOptions} -t ghristov88/wg-manager -t ghristov88/wg-manager:latest-${currentBuild.number} -f Dockerfile .")
                                    .RemoteExecuteAfterDeploy("echo ''")
                                    .RemoteExecuteAfterDeploy("time docker-compose -f docker-compose.yml up -d")
                            })
                    )
            }
        }
        else if (ps.IsMainBranch()) {
            if (ps.IsMainBuildId()) {

            }
        }
    })

genericPipelineV1.run(pipelineSettings)