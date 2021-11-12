#Howto use a private nuget-feed from Azure DevOps Server

When you want to consume a private nuget-feed from Azure DevOps Server there are a few pitfalls you might come aross. This is especially true, if you are working with docker, visual studio and azure pipelines, as all of these have special requirements regarding private nuget-feeds.

##Visual Studio
This is quiet easy and well described here: https://docs.microsoft.com/en-us/azure/devops/pipelines/packages/nuget-restore?view=azure-devops#restore-packages-with-nuget-cli

You just need to make sure, that you have a nuget.config with the URL of your private nuget-feed (no password) that is in the same folder as your solution or project file. Visual Studio will pick that file and if neccessary asks you for your credentials to your Azure DevOps Server to access the feed.
You can add the nuget.config to your repository as it doesn't contain any credentials and your colleagues are also able to restore after they fetch the next time.

##Azure pipeline
When you want to consume a private feed within your Azure build-pipeline you don't have a user that can enter his credentials as with Visual Studio (unattended).
And as you don't want to save your credentials to the nuget.config there is a personal access token (PAT) you can use (https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page). With a PAT you don't expose your credentials and you can set the access rights very fain grained.
For accessing you private feed, just add _read_ privileges in the are of _packaging_ to your PAT. You can then use the PAT eighter as _packageSourceCredentials_ in the nuget.config directly (https://mallibone.com/post/private-nuget-feed-azure-devops), or you can add them on the fly as a task in your pipeline.

<script src="https://gist.github.com/1234Georg/313b70acc629f43fd9dcab9a74bffb7e.js"></script>

{% gist 313b70acc629f43fd9dcab9a74bffb7e %}

With ```dotnet nuget update``` you change one entry in your nuget.config. Just make sure the PAT is added in the ```-p``` paramter. The ```-u``` parameter is not important.

You should be aware of that the PAT will be written in the nuget.config on the build agent in clear text. So this might be a seurity threat.

##Dockerized Build
In a dockerized build you do it somehow similar to azure pipelines. Just copy your project files and your nuget.config to your docker-image. Then you add the PAT to your nuget.config and do a restore.

{% gist 823a897f7f6d9d632eafdf4d688c26ce %}

As above this will add your PAT in plain text to your image which could be a problem. To overcome this the example uses a multi-stage build so that your nuget.config doesn't end up in your shipped docker-image.

##Suggested Microsoft Solution
https://github.com/dotnet/dotnet-docker/blob/main/documentation/scenarios/nuget-credentials.md#using-the-azure-artifact-credential-provider

https://github.com/microsoft/artifacts-credprovider/issues/63#issuecomment-471796987

https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page
https://github.com/dotnet/dotnet-docker/blob/main/documentation/scenarios/nuget-credentials.md