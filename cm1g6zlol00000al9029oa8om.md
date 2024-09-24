---
title: "Joining the Disjoint Tools: A Step-by-Step Guide to Integrate Jenkins and Slack"
seoTitle: "Integrating Slack with Jenkins: Step-by-Step Guide"
seoDescription: "Integrate Slack with Jenkins for CI/CD, enhancing developer productivity. Create custom Slack bots for interactive deployment pipelines"
datePublished: Tue Sep 24 2024 08:46:30 GMT+0000 (Coordinated Universal Time)
cuid: cm1g6zlol00000al9029oa8om
slug: integrate-jenkins-and-slack
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1726663344033/4f8a8a2d-4cfe-4484-8917-254884468ac6.png
tags: productivity, slack, devops, jenkins, developer-tools, devex

---

One of the biggest challenges to developer productivity is context switching, which is caused by poorly integrated CI/CD, source control, task trackers, and observability tools.

Some systems use plugins for integration, but it's often limited. Sometimes, important features are missing, causing repeated switching between systems and manual work.

Fortunately, most of these systems have APIs, which let us create custom integrations to fit our needs. Over the years in engineering, I've found that Slack Messenger is often the best place for building automation. It has a comprehensive API and SDK, allowing the creation of custom UIs and commands.

In this article, we will leverage the official [Slack Bolt](https://api.slack.com/bolt) SDK to manage the execution of all kinds of Jenkins pipelines from Slack.

Here is a demo of one of the scenarios to give you an idea of what we'll implement. The user sends a command to Slack, selects a pipeline, and Slack displays a form with the pipeline's parameters. The user fills in the parameters and runs the pipeline by clicking the "Run" button.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif)

You can find the complete source from this article [here on GitHub](https://github.com/fshchudlo/jenkins-slack-connector).

I used TypeScript, but Slack [also offers SDKs for Java and Python](https://slack.dev/).

# Preparing the Environment

### Registering a new Slack application

First, we need to register a new application in Slack. I recommend creating a separate Slack workspace for testing purposes and following the ["Create an app" section](https://slack.dev/bolt-js/getting-started/#create-an-app) in the Slack Bolt getting started guide.

To make things easier, you can import [a prepared manifest file](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/.assets/slack_app_manifest.yml) during configuration. It configures the minimal required scopes, enables event handling, and registers the required Slack command.

After registering and installing the app into the Slack workspace, we need to [obtain two tokens](https://slack.dev/bolt-js/getting-started/#tokens-and-installing-apps):

* **App Token** can be generated on the **Basic Information** page.
    
* **Bot Token** can be obtained from the **OAuth & Permissions** page.
    

### Bootstrapping an Application Repository

You can simply clone [my repository](https://github.com/fshchudlo/jenkins-slack-connector) and follow the README.md. Alternatively, you can continue following the [Slack Bolt Getting Started guide](https://slack.dev/bolt-js/getting-started/#setting-up-your-project) to bootstrap the application.

First, we need to install some npm dependencies:

`npm install @slack/bolt axios dotenv ts-node typescript`

Next, we need to set up the Slack Bolt app in `app.ts` file:

```typescript
import {App} from "@slack/bolt";
import {AppConfig} from "./app.config";

const app = new App({
    token: AppConfig.SLACK_BOT_TOKEN,
    appToken: AppConfig.SLACK_APP_TOKEN,
    socketMode: true
});

(async () => {
    await app.start(AppConfig.SLACK_BOT_PORT);
    console.log("⚡️ Jenkins-Slack connector is running!");
})();
```

The `AppConfig` used above is just a simple wrapper around `process.env`:

```typescript
import "dotenv/config";

export const AppConfig = {
    SLACK_BOT_TOKEN: process.env.SLACK_BOT_TOKEN,
    SLACK_APP_TOKEN: process.env.SLACK_APP_TOKEN,
    SLACK_BOT_PORT: process.env.SLACK_BOT_PORT,
    JENKINS_URL: process.env.JENKINS_URL,
    JENKINS_CREDENTIALS: {
        username: process.env.JENKINS_API_USER,
        password: process.env.JENKINS_API_TOKEN
    }
};
```

To complete the configuration, export the Slack tokens from the previous step or place them into the `.env` file. The `SLACK_APP_TOKEN` variable is for the App token (starts with `xapp-`), while the `SLACK_BOT_TOKEN` is for the Bot token (starts with `xoxb-`).

### Preparing Jenkins

First, we need to get the Jenkins token to access its API:

1. Log in to Jenkins.
    
2. Click your name in the upper-right corner.
    
3. Click the **Configure** section in the left-hand menu.
    
4. Click the **Add New Token** button.
    
5. Copy the generated token.
    
6. Put the generated token and your login into the `JENKINS_API_TOKEN` and `JENKINS_API_USER` variables. Also, put your Jenkins base URL to the `JENKINS_URL` variable.
    

Now, we need some pipelines to play. You can simply add these [three sample pipelines](https://github.com/fshchudlo/jenkins-slack-connector/tree/main/.assets/sample-pipelines) to your Jenkins.

So, we're ready to write some code!

# Creating the First Slack Command

The [manifest file](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/.assets/slack_app_manifest.yml) I provided before adds the `/run_jenkins_pipeline` command.

Let's add the handler for this command and display the form to choose a Jenkins pipeline we want to run.

```typescript
app.command("/run_jenkins_pipeline", async ({ack, respond}) => {
    await ack();

    const blocks = [
        section("Please, choose pipeline to run"),
        divider(),
        pipelinesDropdown(),
        actionsBlock("submit", [
            button("Run", ActionKeys.RUN_JENKINS_PIPELINE),
            cancelButton()
        ])
    ];

    await respond({blocks: blocks});
});
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Slack forms are built with <a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder" style="pointer-events: none">Slack Block Kit</a>. It's quite verbose, so I created builder functions like <code>divider</code>, <code>button</code>, and <code>cancelButton</code> to make the code compact. <a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder#%7B%22blocks%22:%5B%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22Please,%20choose%20pipeline%20to%20run%22%7D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22input%22,%22label%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Select%20an%20item%22%7D,%22element%22:%7B%22type%22:%22static_select%22,%22options%22:%5B%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Parameterless%20pipeline%20example%22%7D,%22value%22:%22job/parameterless-pipeline-example%22%7D,%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Parameterized%20pipeline%20example%22%7D,%22value%22:%22job/parameterized-pipeline-example%22%7D,%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Interactive%20pipeline%20example%22%7D,%22value%22:%22job/interactive-pipeline-example%22%7D%5D,%22action_id%22:%22pipeline_name%22%7D%7D,%7B%22type%22:%22actions%22,%22block_id%22:%22submit%22,%22elements%22:%5B%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Run%22%7D,%22style%22:%22primary%22,%22value%22:%22run_jenkins_pipeline%22,%22action_id%22:%22run_jenkins_pipeline%22%7D,%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Cancel%22%7D,%22style%22:%22danger%22,%22value%22:%22display_cancellation_message%22,%22action_id%22:%22display_cancellation_message%22%7D%5D%7D%5D%7D" style="pointer-events: none">Here is our form declaration</a> in pure Block Kit syntax. The link is&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder" style="pointer-events: none">Slack Block Kit Builder</a>, which I recommend you use to simplify form creation.</div>
</div>

The code above registers the [command handler](https://slack.dev/bolt-js/concepts/commands) that does the following:

* Calls the `ack()` function to notify Slack that our app can handle this command.
    
* Builds a form with a header, a dropdown to choose a pipeline to run, and the **Run** and **Cancel** buttons.
    
* Renders the form to the user by calling the`respond` function.
    

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Slack API offers much more than just the <code>ack</code> and <code>respond</code> functions. The&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://slack.dev/bolt-js/reference/#listener-function-arguments" style="pointer-events: none">Listener function arguments</a>&nbsp;section lists all the available options.</div>
</div>

Here is what will happen if we run our bot and type `/run_jenkins_pipeline` in Slack:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788422626/b3a510fd-db83-4329-8c37-a7ee78c5e685.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788422626/b3a510fd-db83-4329-8c37-a7ee78c5e685.gif)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">We use hardcoded pipeline names for simplicity. However, you can implement advanced scenarios using <a target="_blank" rel="noopener noreferrer nofollow" href="https://api.slack.com/reference/block-kit/block-elements#select" style="pointer-events: none">Slack Select</a> controls that support paging, filtering, dynamic loading, and more. To explore what the Jenkins API offers, simply add <code>/api</code> to the URL of the page from which you want to get data. That opens the documentation for that specific API endpoint.</div>
</div>

So, our bot can display a form in response to a command. Let's teach the bot to run the Jenkins pipeline on the form submission.

# Running a Parameterless Pipeline

We already added the **Run** and **Cancel** buttons, but they do nothing currently.

To fix that, we need to use [action handlers](https://slack.dev/bolt-js/concepts/action-respond). The **Cancel** button handler is pretty straightforward:

```typescript
app.action(ActionKeys.DISPLAY_CANCELLATION_MESSAGE, async ({ack, respond}) => {
    await ack();
    await respond("Ok, I've canceled the operation. See you soon.");
});
```

Now, let's look to the **Run** button handler:

```typescript
app.action(ActionKeys.RUN_JENKINS_PIPELINE, async ({context, body, ack, respond}: AllMiddlewareArgs & SlackActionMiddlewareArgs) => {
    await ack();
    const selectedPipelineOption = body.state.values.pipeline_name.pipeline_name.selected_option;

    try {
        const startedJobUrl = await context.jenkinsAPI.startJob(selectedPipelineOption.value);
        await respond(`:rocket: Here is the started ${link(pipelineDisplayName, startedJobUrl)} pipeline`);
    } catch (error) {
        await respond({blocks: errorMessage(`Unable to run \`${pipelineDisplayName}\` pipeline`, error)});
    }
});
```

First, we call already familiar `ack`, then we get the selected pipeline from `body.state.values` and call `JenkinsAPI.startJob` to run the specified pipeline. Finally, we respond to the user with a link to the started job or report an error.

Here is the `JenkinsAPI.startJob` implementation, which is a bit tricky:

```typescript
export class JenkinsAPI {
    private readonly auth: BasicAuthCredentials;
    private readonly jenkinsUrl: string;

    constructor(auth: BasicAuthCredentials, jenkinsUrl: string) {
        this.auth = auth;
        this.jenkinsUrl = jenkinsUrl;
    }

    async startJob(jobName: string, jobParams?: any): Promise<string> {
        const runJobUrl = `${this.jenkinsUrl}/${jobName}/build`;

        const queuedJobUrl = (await this.request(runJobUrl, "POST", jobParams)).headers.location;

        // give Jenkins time to start the job
        await new Promise(resolve => setTimeout(resolve, 300));

        const startedJob = (await this.request(`${queuedJobUrl}/api/json`, "GET")).data;

        if (startedJob.blocked) {
            return Promise.reject(startedJob.why);
        }
        return startedJob.executable.url;
    }
    private async request(url: string, method: Method = "GET", params?: any): Promise<AxiosResponse> {
        return axios.request({
            method,
            url,
            params,
            auth: this.auth,
            headers: {"accept": "application/json", "accept-encoding": "gzip, deflate, br"}
        });
    }
}
```

This method sends a **POST** request to the "/build" API to start the pipeline and gets the **queued** job URL from the response `location` header. After that, we request the queued job URL, hoping to get a **running** job URL from the `queuedJobInfo.executable.url` property.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This strange&nbsp;<strong>setTimeout&nbsp;</strong>call is placed in the function to give Jenkins time to start the queued job. This is an obvious <a target="_blank" rel="noopener noreferrer nofollow" href="https://www.pluralsight.com/tech-blog/forms-of-temporal-coupling/" style="pointer-events: none">temporal coupling</a> issue, but unfortunately, we can't avoid it here.</div>
</div>

Sometimes, Jenkins is not able to run the job. For example, because there is no available build agent or because the [concurrent execution is disabled](https://www.jenkins.io/doc/book/pipeline/syntax/#options) and there is a job already running. In that case, we raise an error with the `queuedJobInfo.why` description to display an error message to the user.

So, let's see how it looks in action:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789052807/72804be2-a5fc-44e1-889b-8a0328c8d5a4.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789052807/72804be2-a5fc-44e1-889b-8a0328c8d5a4.gif)

As you can see, we also received a message about the pipeline completion. This is implemented on the Jenkins side using the [Jenkins Slack Notification plugin](https://www.jenkins.io/doc/pipeline/steps/slack/) and the following code:

```typescript
  pipeline {
    ...
    post {
        always {
            script {
                def authorId = slackUserIdFromEmail(currentBuild.rawBuild.getCause(Cause.UserIdCause)?.getUserId())
                if (authorId) {
                    def message = currentBuild.result == 'SUCCESS' ? ":white_check_mark: `${env.JOB_BASE_NAME}` build <${env.BUILD_URL}|completed successfully>" : ":octagonal_sign: `${env.JOB_BASE_NAME}` build <${env.BUILD_URL}|had been failed>"
                    slackSend(message: message, sendAsText: true, channel: authorId, botUser: true, tokenCredentialId: 'jenkins-slack-connector-bot-token')
                }
            }
        }
    }
}
```

Here, we get the Slack user ID from the build starter's email and send a message. We also use the `botUser` and `tokenCredentialId` parameters to specify our bot token. This way, the user will see all the messages in the same Slack conversation.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The bot is designed so you can send the <code>run_jenkins_pipeline</code> command to any channel or individual messages with other people. The bot responds with <a target="_blank" rel="noopener noreferrer nofollow" href="https://api.slack.com/surfaces/messages#ephemeral" style="pointer-events: none">ephemeral messages</a>, which only you can see and won't disturb others. Meanwhile, Jenkins will always send notifications to you in the bot chat.</div>
</div>

# Running Pipelines From Multiple Users

Currently, we use a single token to access the Jenkins API. If we share our bot with the entire team, everyone will run pipelines from this user, and all messages will come to him instead of teammates. This is far from what we want.

To solve this, each user should specify their Jenkins token. We can use [Slack middleware](https://slack.dev/bolt-js/concepts/listener-middleware) for that: if a called listener needs access to Jenkins API, we check if the user provided Jenkins credentials. If not, a modal form prompts them to provide and save a token.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788103293/fb003f32-54d4-4613-a086-44019832ba21.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788103293/fb003f32-54d4-4613-a086-44019832ba21.gif)

Here is the code of the middleware:

```typescript
export async function constructJenkinsAPI({context, client, body, next}) {
    const credentials = CredentialsStore.getCredentials(context.userId);
    if (credentials) {
        context.jenkinsAPI = new JenkinsAPI(credentials, AppConfig.JENKINS_URL);
        await next();
        return;
    }
    await client.views.open({
        trigger_id: body.trigger_id,
        view: buildJenkinsTokenModalForm()
    });
    await next();
}

function buildJenkinsTokenModalForm(): ModalView {
    return {
        type: "modal",
        callback_id: ActionKeys.SAVE_ACCESS_TOKENS,
        title: {
            type: "plain_text",
            text: "Access tokens required"
        },
        submit: {
            type: "plain_text",
            text: "Submit",
            emoji: true
        },
        blocks: [
            section("Hi! Please, give me access token to perform useful stuff for you"),
            divider(),
            textbox("Jenkins API token", "jenkins_token")
        ]
    };
}
```

This middleware checks whether the user has Jenkins credentials. If not, the middleware displays a [modal window](https://slack.dev/bolt-js/concepts/creating-modals) requesting the user to provide a token. If the user already has saved credentials, the middleware creates a **JenkinsAPI** instance and adds it to the Slack Bolt `context`, which [can be used to enrich requests with additional information](https://tools.slack.dev/bolt-js/concepts/context/). Thus, instead of constructing a **JenkinsAPI** instance in each listener, we use the instance from the `context`.

To register that middleware, we should add it as a second parameter to each of our handler registrations:

```typescript
app.command("/run_jenkins_pipeline", constructJenkinsAPI, async ({ack, respond}) => {
...
});

app.action(ActionKeys.RUN_JENKINS_PIPELINE, constructJenkinsAPI, async ({context, body, ack, respond}) => {
...
});
```

The next piece is the [Slack view listener](https://slack.dev/bolt-js/concepts/view-submissions) that handles dialog window submission and stores the credentials:

```typescript
app.view(ActionKeys.SAVE_ACCESS_TOKENS, constructUser, async ({ack, body, context})=> {
    const respond = async (title: string, ...blocks: any[]): Promise<void> => {
        await ack({
            response_action: "update",
            view: {
                type: "modal",
                title: {type: "plain_text", text: title},
                blocks: blocks
            }
        });
    };

    const payload: any = getSlackFormValues(body.view.state.values);
    const userInfo = (await client.users.info({user: context.userId})).user;

    CredentialsStore.saveCredentials(context.userId, {username: userInfo.profile.email, password: payload.jenkins_token});

    await respond("Success", section(":thumbsup: Token was saved. Now you can use any of my features"));
});
```

Let's also look to the `CredentialsStore` used above. It simply saves tokens in memory:

```typescript
import {BasicAuthCredentials} from "./jenkins-api/BasicAuthCredentials";

const CREDENTIALS_STORE: Record<string, BasicAuthCredentials> = {}

export class CredentialsStore {
    static saveCredentials(userId: string, credentials: BasicAuthCredentials): void {
        CREDENTIALS_STORE[userId] = credentials;
    }

    static getCredentials(userId: string): BasicAuthCredentials | null {
        if (userId in CREDENTIALS_STORE) {
            return CREDENTIALS_STORE[userId];
        }
        return null;
    }
}
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Of course, in-memory implementation is applicable only for demonstration since tokens will be lost after each application restart. I encourage you to thoughtfully consider how to store tokens securely in your environment and add the required implementation on your own. For example, a good option is to use secret storage like Hashicorp Vault combined with an established secret rerolling policy.</div>
</div>

So, now we can run Jenkins pipelines as different users. Let's implement more feature use cases!

# Running a Parameterized Pipeline

Let's modify our pipeline running handler to run parameterized pipelines, too.

Here is the listing of the new action implementation:

```typescript
app.action(ActionKeys.RUN_JENKINS_PIPELINE, async ({body, ack, respond}: AllMiddlewareArgs & SlackActionMiddlewareArgs) => {
    ...
    const pipelineParameters = await jenkinsAPI.getJobParameters(pipelineId);

    if (pipelineParameters) {
        const parametersForm = buildPipelineParametersForm(pipelineParameters, pipelineDisplayName, pipelineId);        
        await respond({blocks: parametersForm});
    } else {
        await runParameterlessPipeline(context.jenkinsAPI, pipelineId, pipelineDisplayName, respond);
    }
});
```

We start with calling `jenkinsAPI.getJobParameters` method. Here is it's implementation:

```typescript
export class JenkinsAPI {
    ...
    async getJobParameters(jobPath: string): Promise<JenkinsParameterDefinition[] | null> {
        const url = `${this.jenkinsUrl}/${jobPath}/${JENKINS_API_POSTFIX}`;

        const jobInfo = (await this.request(url, "GET")).data;
        const parametersDefinition = jobInfo.property.find((item: object) => "parameterDefinitions" in item);

        return parametersDefinition?.parameterDefinitions || null;
    }
}
```

If the pipeline doesn't have parameters, we call the `runParameterlessPipeline` function that does the same as we implemented previously:

```typescript
async function runParameterlessPipeline(jenkinsAPI: JenkinsAPI, pipelineId: string, pipelineDisplayName: string, respond: RespondFn) {
    try {
        const startedJobUrl = await jenkinsAPI.startJob(pipelineId);
        await respond(`:rocket: Here is the ${link(pipelineDisplayName, startedJobUrl)} pipeline that you run`);
    } catch (error) {
        await respond({blocks: errorMessage("Unable to run pipeline", error)});
    }
}
```

However, if the pipeline is parameterized, we call the `buildPipelineParametersForm` function to create a parameters form and display it to the user:

```typescript
function buildPipelineParametersForm(pipelineParameters: JenkinsParameterDefinition[], pipelineDisplayName: string, pipelineId: string) {
    return [
        section("Please, specify parameters"),
        divider(),

        ...mapJenkinsParametersToSlackControls(pipelineParameters),

        actionsBlock(pipelineId, [
            button("Run", ActionKeys.RUN_PARAMETERIZED_JENKINS_PIPELINE, pipelineDisplayName),
            cancelButton()
        ])
    ];
}
```

The `mapJenkinsParametersToSlackControls` function is a bit complex because it humanizes parameter names, converts HTML to Slack markup, and more. While the full implementation is [in the repository](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/actions/helpers/mapJenkinsParametersToSlackControls.ts), here is a simplified version. As you can see, essentially, this is a simple switch case mapping:

```typescript
function mapJenkinsParametersToSlackControls(parametersDefinition) {
    return parametersDefinition.map(parameter => {
        const controlName = humanizeParameterName(parameter.name);

        switch (parameter.type) {
            case "StringParameterDefinition":
                return textbox(controlName, parameter.name, parameter.initialValue);
            case "TextParameterDefinition":
                return textarea(controlName, parameter.name, parameter.initialValue);
            case "ChoiceParameterDefinition":
                return staticSelect(parameter.name, getSelectOptions(parameter), parameter.name);
            case "BooleanParameterDefinition":
                const initialOption = {[parameter.name]: humanizeParameterName(parameter.name)};
                return checkboxGroup(" ", parameter.name, initialOption, parameter.initialValue ? initialOption : undefined);
            default:
                throw new Error(`Unknown control type ${parameter.type} of the field ${parameter.name}`);
        }
    });
}
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This switch case is a classic example of violating the <a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle" style="pointer-events: none">Open-Closed Principle</a>. I did this to keep the code readable for the article. However, if you need to implement many controls in your app, reworking this code to use the&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Strategy_pattern" style="pointer-events: none">Strategy</a>&nbsp;or&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Command_pattern" style="pointer-events: none">Command</a>&nbsp;pattern will be worth it.</div>
</div>

The last step to complete the use case is adding an action handler to run a parameterized pipeline after submitting the parameters form. Here, we get the submitted values and pass them to the `jenkinsAPI.startJob` method:

```typescript
app.action(ActionKeys.RUN_PARAMETERIZED_JENKINS_PIPELINE, async ({context, body, payload, ack, respond}) => {
    ack();

    const pipelineId = payload.block_id;
    const submittedValues = getSlackFormValues(body.state.values);

    try {
        const startedJobUrl = await context.jenkinsAPI.startJob(pipelineId, submittedValues);
        await respond(`:rocket: Here is the ${link(payload.value, startedJobUrl)} pipeline that you run`);
    } catch (error) {
        respond({blocks: errorMessage("Unable to run pipeline", error)});
    }
});
```

Let's look at the complete use case in action:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif)

Great! Let's take the final step and teach our bot to work with interactive pipelines.

# Adding Interactivity With Jenkins Input Step

[Jenkins Input Step](https://www.jenkins.io/doc/pipeline/steps/pipeline-input-step/) is a very nice feature that allows you to ask for user input during pipeline execution and create interactive pipelines. If you have never seen it in action, here is how it looks in Jenkins UI:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725636025169/99a7db4a-d4f6-4a28-bb42-9e2fdae77f84.png align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725636025169/99a7db4a-d4f6-4a28-bb42-9e2fdae77f84.png)

I found the Input Step feature especially useful when developing a new delivery pipeline for my project. Thanks to it, my team and I now have a fully automated and interactive delivery process that asks whether to send release notes and what Jira tasks to mention, what environments to roll out the update to, which of our 15 services to deploy, and in what order, and whether to roll back to a previous version if the deployment was unsuccessful.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">A sudden benefit of the Slack bot I mentioned is that the Jenkins UI for Input Requests has broken a couple of times during Jenkins upgrades. At the same time, the bot has continued to work well since it leverages only input API.</div>
</div>

So, let's integrate this awesome feature into Slack, too!

### 1\. Declaring Input Step in the Pipeline

First, let's see how the Input Step is defined in the Jenkins pipeline:

```typescript
pipeline {
...
stage('Test stage 2') {
  steps {
      script {
         input(
            id: 'some-approval',
            message: 'Please, choose parameters and press "Continue"',
            parameters: [
                stringParam(name: 'someStringParam', defaultValue: '', description: 'Some string popup param'),
                booleanParam(name: 'someBooleanPopupParam', description: 'Some boolean popup param')
            ],
            ok: 'Continue')
    }
  }
}
...
}
```

Note that we explicitly pass the unique `id` parameter. If the pipeline has multiple input steps, this prevents submitting a form for the first input while the pipeline has already requested the second one.

### 2\. Notifying Slack Bot About Input Request

We should somehow notify the Slack bot that the pipeline requested user input. We will do that by sending a Slack message. Even though we will intercept this message in the next step, I prefer to make it human-readable. This will work as a fallback, allowing the user to follow the link and provide input with Jenkins UI ​​even if the bot fails to complete its task.

```typescript
def sendInputRequestToTheSlack() {
    def authorId = slackUserIdFromEmail(currentBuild.rawBuild.getCause(Cause.UserIdCause)?.getUserId())
    if (authorId == null) {
        return
    }

    def message = ":keyboard: <${env.BUILD_URL}|${env.JOB_BASE_NAME}> requested some input. <${env.RUN_DISPLAY_URL}|Please, provide it here>"

    slackSend(message: message, sendAsText: true, channel: authorId, botUser: true, tokenCredentialId: 'jenkins-slack-connector-bot-token')
}
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">We should call the <code>sendInputRequestToTheSlack</code> function <strong>before</strong> calling the Input Step because Jenkins stops execution until the user gives input. Again, this causes a temporal coupling issue because the Slack listener might start the notification handling before Jenkins shows the input form. The solution is the same as before - use <code>setTimeout</code> on the listener side.</div>
</div>

Now, we need to configure the Slack bot to listen to messages.

First, we should disable the `ignoreSelf` option so the bot can listen to its own messages:

```javascript
const app = new App({
    token: AppConfig.SLACK_BOT_TOKEN,
    appToken: AppConfig.SLACK_APP_TOKEN,
    socketMode: true,
    ignoreSelf: false
});
```

We have already enabled event listening with the [manifest file](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/.assets/slack_app_manifest.yml). It’s disabled by default since Slack generates tons of events.

```yaml
settings:
  event_subscriptions:
    bot_events:
      - message.im
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><code>message.im</code> event works only for direct messages. If you want to broadcast Jennkins input requests to channels so any teammate can respond, add the <code>message.channels</code> event to that manifest section.</div>
</div>

### 3\. Listen to the Notification in the Slack Bot

Now, we're ready to implement the [message listener](https://api.slack.com/events/message):

```typescript
app.message(async ({context, message, client}) => {
    if (context.botId != message.bot_id && message.text.indexOf(":keyboard:")!==0) {
        return;
    }
    const [{url: buildUrl, text: buildName}, {url: buildDisplayUrl}] = message.blocks.flatMap(b => b.elements).flatMap(e => e.elements).filter(e => e.type == "link");

    // give Jenkins time to render the input
    await new Promise(resolve => setTimeout(resolve, 300));
    
    const jenkinsInputDefinition = await context.jenkinsAPI.getJobPendingInput(buildUrl);
    const formBlocks = jenkinsInputDefinition? await buildJenkinsPendingInputForm(buildUrl, jenkinsInputDefinition) : buildInputHandlingFailureForm(buildName, buildUrl);

    await client.chat.update({
        channel: message.channel,
        ts: message.ts,
        blocks: formBlocks,
    });
});
```

This code does the following:

* It checks if the message is from the bot and starts with a ⌨️ emoticon. This way, it only handles messages about Jenkins Input Requests.
    
* It extracts the running job's URL and name from the message blocks.
    
* It fetches the pending input definition by calling the `jenkinsAPI.getJobPendingInput` , which is very similar to already known `jenkinsAPI.getJobParameters`.
    
* If the input definition is empty, someone has already submitted input, or a timeout was reached. In that case, it shows the error form by calling `buildInputHandlingFailureForm` function.
    
* Otherwise, we show the parameters form by calling the `buildJenkinsPendingInputForm` function. This maps Jenkins parameters to Slack controls, as we saw in the “Running Parameterized Pipeline” section.
    

### 4\. Handling Input Form Submit

We're almost done. All that's left is to submit or cancel pending input depending on which button the user clicks on the rendered input form. Here is the submit handler:

```typescript
app.action(ActionKeys.SUBMIT_PENDING_JENKINS_INPUT, async ({context, body, ack, respond}) =>{
    ack();

    const submitUrl = body.actions[0].value;
    const buildUrl = body.actions[0].block_id;
    
    const submittedValues = getSlackFormValues(body.state.values);

    try {
        await context.jenkinsAPI.submitJobPendingInput(buildUrl, submitUrl, submittedValues);

        await respond(`Input was submitted to the ${link("build", buildUrl)} by ${body.user_name}`);
    } catch (error) {
        await respond({blocks: errorMessage(`Unable to submit input for the ${link("build", buildUrl)}`, error)});
    }
});
```

As you can see, it is almost identical to the handler we implemented to run the parameterized pipeline. The only difference is that we call the `jenkinsAPI.submitJobPendingInput` instead of the `jenkinsAPI.startJob`.

Here is the listing of the `jenkinsAPI.submitJobPendingInput` method:

```typescript
export class JenkinsAPI {
    ...
    async submitJobPendingInput(buildUrl: string, submitUrl: string, formValues: any): Promise<void> {
        const inputDefinition = await this.getJobPendingInput(buildUrl);

        if (!inputDefinition || !submitUrl.includes(`inputSubmit?inputId=${inputDefinition.id}`)) {
            throw new Error("Input was already submitted or timed out");
        }

        const convertedValues = Object.entries(formValues).map(([key, value]) => ({name: key, value}));
        await this.request(`${this.jenkinsUrl}${submitUrl}`, "POST", {json: JSON.stringify({parameter: convertedValues})});
    }
}
```

We request pending input from Jenkins again to ensure that the input is still expected and its ID matches the submitted input's ID. Then, we convert the submitted values to the Jenkins API format and post them.

### 5\. Handling Input Request Abortion

The last step is to handle input abortion. Here is the code of the handler:

```typescript
app.action(ActionKeys.ABORT_PENDING_JENKINS_INPUT, await ({ context, body, ack, respond }) => {
    await ack();

    const abortUrl = body.actions[0].value;

    try {
        await context.jenkinsAPI.abortJobPendingInput(abortUrl);
        await respond(`Input was aborted by ${body.user_name}`);
    } catch (error) {
        await respond({ blocks: errorMessage("Input abort failed", error) });
    }
};
```

Here, we simply call the `jenkinsAPI.abortJobPendingInput` method and notify the user about the abortion.

The code of the `jenkinsAPI.abortJobPendingInput` method is very straightforward:

```typescript
 export class JenkinsAPI {
    async abortJobPendingInput(abortUrl: string): Promise<void> {
        await this.request(`${this.jenkinsUrl}/${abortUrl}`, "POST");
    }
}
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">What happens with the running job after input abortion depends on the pipeline implementation. The job will be aborted by default, but it is possible to catch <a target="_blank" rel="noopener noreferrer nofollow" href="https://javadoc.jenkins.io/plugin/workflow-step-api/org/jenkinsci/plugins/workflow/steps/FlowInterruptedException.html" style="pointer-events: none">FlowInterruptedException</a> and implement some custom logic.</div>
</div>

Well, we're finally done! Let's see the magic in action:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789485490/100290b8-399b-4c70-b389-fed9e2859735.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789485490/100290b8-399b-4c70-b389-fed9e2859735.gif)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The initial message stays on the screen long before the Slack bot updates it, which can look awkward. Usually, this isn't a problem because users don't watch the screen constantly. But if it bothers you, you can change the <code>client.chat.update</code> call to <code>client.chat.postMessage({channel, blocks})</code>. This way, messages will be sent one after another.</div>
</div>

Well, we implemented the latest use case!

Generally, we've covered many features of the Slack Bolt SDK and built a comprehensive automation. However, to keep the article easy to read, I skipped error handling, monitoring, logging, security, and testing (but you can check [the source code](https://github.com/fshchudlo/jenkins-slack-connector) for tests).

# Wrapping Up

Integrating Slack with other tools can significantly enhance your development workflow. Its true power lies in connecting multiple systems. Imagine starting your workday with a Slack bot listing your possible activities like:

* Check your recently closed tasks and offers post-actions, like updating documentation.
    
* Display pull requests you participated in that haven't been merged yet.
    
* Shows your next task and its recent updates.
    
* List your branches without pull requests, indicating abandoned work or forgotten experiments.
    
* Show recent findings from automated security scans.
    
* Highlight unresolved Slack threads in the support channel with questions for your team.
    
* Show unresolved support tickets if their count is higher than usual and offer you help your teammate on call.
    

Such a cohesive and automated environment can greatly improve productivity and collaboration. I hope this guide will help you create a dev environment that meets the unique needs of your team.

---

Life is so beautiful,

Fedor