---

title: SonarQube for mixed .NET and JavaScript/TypeScript projects
excerpt: "SonarQube always had a bad reputation when it comes to repositories that contain both the front-end as well as the back-end code. Here are two approaches."

tags:
- alm
- tool
- build-pipeline
- sonarqube
---

**Why should I care?**  
As a long-time maintainer of [coding guidelines for C#](https://csharpcodingguidelines.com/), it should not come as a surprise that I believe tools like [SonarQube](https://www.sonarqube.org/) can really help you improve the quality and maintainability of your C#, JavaScript and TypeScript code. It doesn't give you the fast feedback loop that you get from local development tools like [ESLint](https://eslint.org/) and .NET Static Analysis, but I've lost count of the many times it saved us from serious production issues. However, SonarQube always had a bad reputation when it comes to repositories that contain both the front-end as well as the back-end code. For example, for us, it's pretty common to have the .NET APIs shipped with the React/TypeScript SPA as a single unit. Well, I've done some investigations recently and figured out there are practically two approaches.

**Single repository with a single SonarQube project**  
This approach requires the front-end code to be part of the .NET solution file, and thus works best for teams where front-end developers are fine to use the .NET IDE, or where all developers are full-stack developers. It will give you a single report for the entire repo, for example, through a status check on code repositories like Github. The flipside is that somehow, it won't tell you the total number of tests across the different languages. 

Here are the high-level steps to make that work. The exact steps are obviously highly dependent on the specific project.

1. Include your `.js`/`.jsx`/`.ts`/`.tsx` files in the .NET project as content files
2. Setup a SonarQube project with language profiles for C#, JavaScript and/or TypeScript
3. Make sure your build engine has the required Java version installed
4. Add the [SonarScanner for .NET](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/) (`dotnet-sonarscanner`) to your build script of choice
5. Ensure your build script runs the following steps in the specified order
	1. Run `dotnet tool run dotnet-sonarscanner begin` to begin the analysis with parameters instructing it where to pick up the code coverage files from the front-end and back-end code
	1. Build the front-end code and run the tests with coverage enabled using the OpenCover format
	1. Build the `.sln` file using `dotnet` and tell it to output the coverage results, again in OpenCover format
	1. Run `dotnet tool run dotnet-sonarscanner end` to complete the analysis and push the results to SonarQube.

**Single repository but with separate SonarQube projects for front-end and back-end**  
For teams where front-end and back-end developers mostly work in isolation, each using dedicated tooling, but still delivering the entire solution using the same release cadance, the previous option may not work. This applies to most of the teams at ORM as well as the newer developments at EHS. In that case, having two separate SonarQube projects fueled from a single repository works a lot better. This is a relevatively new option in SonarQube and not that well described. That's why I decided to write this blog post. The advantage is that you end up with two nicely separated reports in your Github pull request. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/sonar_pr_result.jpg" class="align-center"/> 

The disadvantage of this is that it's a bit more work to set it up correctly. So here are the high-level steps to get this working.

1. Set-up a frontend SonarQube project with the JavaScript and/or TypeScript quality profiles
1. Under "Project Settings -> General Settings -> Pull Request Decoration", enable the "Mono repository front-end analysis" and point it to the correct repository
1. Add the [sonarqube-scanner](https://www.npmjs.com/package/sonarqube-scanner) NPM package to your front-end packages. It automatically downloads anything that it needs, including Java. 
1. It depends on your build pipeline, but in the IAM project, we've set-up an `npm` task that will tell Node to run the following JavaScript file to invoke the scanner and push the result to the front-end project

	```csharp
	const scanner = require('sonarqube-scanner');

	if (!process.env.onBuildServer) {
		console.log('skipping SonarQube scan of the frontend code');
		return;
	}

	scanner({
		serverUrl: process.env.sonar_host_url,
		token: process.env.sonar_login,
		options: {
		'sonar.projectName': 'MyFrontedSqProject',
		'sonar.projectKey': process.env.sonarqube_frontend_key,
		'sonar.sources': 'features,admin,id,shared',
		'sonar.tests': '__tests__',
		'sonar.javascript.lcov.reportPaths': 'coverage/lcov.info',
		'sonar.javascript.file.suffixes': '.js,.jsx',
		'sonar.typescript.file.suffixes': '.ts,.tsx',
		'sonar.sourceEncoding': 'UTF-8',
		'sonar.pullrequest.key': process.env.npm_config_prNumber,
		'sonar.pullrequest.branch': process.env.npm_config_prBranch,
		'sonar.pullrequest.base': process.env.npm_config_prBaseBranch,
		}
	});
	```

1. Set-up a backend SonarQube project with the C# quality profiles of your choice
1. Under "Project Settings -> General Settings -> Pull Request Decoration", enable the "Mono repo backend-end analysis" and point it to the correct repository
1. Make sure your build engine has the required Java version installed
1. Add the [SonarScanner for .NET](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/) o your build script of choice
1. Ensure your build script runs the following steps in the specified order
	1. Run `dotnet tool run dotnet-sonarscanner begin` to begin the analysis with parameters instructing it where to pick up the code coverage files from the front-end and back-end code
	1. Build the `.sln` file using dotnet and tell it to output the coverage results in OpenCover format
	1. Run `dotnet tool run dotnet-sonarscanner end` to complete the analysis and push the results to the back-end SonarQube project.
		
**Some final words**  
The need for that second more elaborate option is really a necessary evil due to the .NET scanner forcing you to add all your front-end assets to .NET projects. This is a [known limitation](https://jira.sonarsource.com/browse/MMF-1411), so hopefully Sonar will fix this at some point. Also, having a NuGet package that includes Java, just like the NPM version is on my wishlist as well. 

So what do you think about all of this? Do you believe in the value of SonarQube? If so, how do you map your SonarQube projects to your repositories? And if not, what do you do to control the quality of your code? I would love to hear your thoughts by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
