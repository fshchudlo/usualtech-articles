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

One of the biggest blockers to developer productivity is context switching—constantly jumping between CI/CD pipelines, source control, task tracker, and observability tools.

While some platforms offer plugins to bridge the gaps, they’re often limited in scope. Critical features can be missing, forcing developers to toggle between tools and workflows.

Fortunately, most of these tools offer APIs, so we can build custom integrations that align with our needs. Through my engineering experience, I’ve found that Slack is an ideal hub for automating tasks. With its powerful API and SDK, we can create custom UIs, commands, and workflows—all within the chat interface teams already use.

In this article, we’ll leverage the [Slack Bolt SDK](https://api.slack.com/bolt) to manage pipeline executions without leaving the Slack interface.

Here is a preview of one of the scenarios we'll implement. A user sends a Slack command, selects a pipeline, and Slack generates a form to capture pipeline parameters. Once the parameters are filled, the user clicks "Run," and the pipeline is triggered

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725790363593/afb3def0-00ba-4e22-b835-b884332bf433.gif)

The full source code for this project is [available on GitHub](https://github.com/fshchudlo/jenkins-slack-connector), and while I used TypeScript, Slack also supports [Java and Python SDKs](https://slack.dev/) for similar implementations.

# Preparing the Environment

### Registering a new Slack application

To get started, we need to create a new Slack application. I recommend setting up a separate Slack workspace for testing and follow the ["Create an app"](https://slack.dev/bolt-js/getting-started/#create-an-app) section in the Slack Bolt Getting Started guide.

You can import a [prepared manifest file](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/.assets/slack_app_manifest.yml) during configuration to streamline the process. This file will automatically configure the necessary scopes, enable event handling, and register the required Slack command.

Once you've registered and installed the app in your Slack workspace, you’ll need to obtain [two tokens](https://slack.dev/bolt-js/getting-started/#tokens-and-installing-apps):

* **App Token** can be generated on the **Basic Information** page.
    
* **Bot Token** can be obtained from the **OAuth & Permissions** page.
    

### Bootstrapping an Application Repository

You can either [clone my repository](https://github.com/fshchudlo/jenkins-slack-connector) and follow the steps in README.md or continue with the [Slack Bolt guide](https://slack.dev/bolt-js/getting-started/#setting-up-your-project) to bootstrap the app manually.

Start by installing the required npm dependencies:

`npm install @slack/bolt axios dotenv ts-node typescript`

Next, set up the Slack Bolt app in the `app.ts` file. Here’s an example setup:

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

The `AppConfig` is a simple wrapper for environment variables:

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

You'll also need to export the Slack tokens from the previous step or place them in the `.env` file:

* `SLACK_APP_TOKEN` (App token, starts with `xapp-`)
    
* `SLACK_BOT_TOKEN` (Bot token, starts with `xoxb-`)
    

### Preparing Jenkins

Next, we need to configure Jenkins to allow API access. Follow these steps to get the required token:

1. Log in to Jenkins.
    
2. Click your profile name in the upper-right corner.
    
3. Navigate to **Configure** in the left-hand menu.
    
4. Click the **Add New Token** button to generate a new token.
    
5. Copy the token for later use.
    

Now, store the token and your Jenkins login information in environment variables:

* `JENKINS_API_TOKEN` (your generated API token)
    
* `JENKINS_API_USER` (your Jenkins username)
    
* `JENKINS_URL` (base URL of your Jenkins instance)
    

Finally, to test your setup, you can add the following sample pipelines to Jenkins:

Finally, we need some pipelines to play. You can simply add these [three sample pipelines](https://github.com/fshchudlo/jenkins-slack-connector/tree/main/.assets/sample-pipelines) to your Jenkins.

Now that everything is set up, we’re ready to dive into the code!

# Creating the First Slack Command

The [manifest file](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/.assets/slack_app_manifest.yml) we used earlier adds a `/run_jenkins_pipeline` Slack command. Now, let’s write a handler for this command that will display a form allowing users to select and run a Jenkins pipeline.

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
<div data-node-type="callout-text">Slack forms are built using <a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder" style="pointer-events: none">Slack Block Kit</a>, which can be quite verbose. To simplify things, I’ve created helper functions like <code>divider()</code>, <code>button()</code>, and <code>cancelButton()</code> to make the code more compact. If you're new to Block Kit, I recommend using the <a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder" style="pointer-events: none">Slack Block Kit Builder</a> to streamline form creation. <a target="_blank" rel="noopener noreferrer nofollow" href="https://app.slack.com/block-kit-builder#%7B%22blocks%22:%5B%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22Please,%20choose%20pipeline%20to%20run%22%7D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22input%22,%22label%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Select%20an%20item%22%7D,%22element%22:%7B%22type%22:%22static_select%22,%22options%22:%5B%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Parameterless%20pipeline%20example%22%7D,%22value%22:%22job/parameterless-pipeline-example%22%7D,%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Parameterized%20pipeline%20example%22%7D,%22value%22:%22job/parameterized-pipeline-example%22%7D,%7B%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Interactive%20pipeline%20example%22%7D,%22value%22:%22job/interactive-pipeline-example%22%7D%5D,%22action_id%22:%22pipeline_name%22%7D%7D,%7B%22type%22:%22actions%22,%22block_id%22:%22submit%22,%22elements%22:%5B%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Run%22%7D,%22style%22:%22primary%22,%22value%22:%22run_jenkins_pipeline%22,%22action_id%22:%22run_jenkins_pipeline%22%7D,%7B%22type%22:%22button%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Cancel%22%7D,%22style%22:%22danger%22,%22value%22:%22display_cancellation_message%22,%22action_id%22:%22display_cancellation_message%22%7D%5D%7D%5D%7D" style="pointer-events: none">Here is the preview of our form</a> in pure Block Kit syntax</div>
</div>

The code snippet below registers a [command handler](https://slack.dev/bolt-js/concepts/commands) for the `/run_jenkins_pipeline` command and performs the following steps:

* Calls the `ack()` to let Slack know that our app is processing the command.
    
* Builds a form with a header, a dropdown menu to select a Jenkins pipeline, and two buttons: **Run** and **Cancel**.
    
* Sends form back to Slack by calling the `respond()` function, allowing the user to interact with it.
    

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Slack API offers much more than just the <code>ack</code> and <code>respond</code> functions. Look at the <a target="_blank" rel="noopener noreferrer nofollow" href="https://slack.dev/bolt-js/reference/#listener-function-arguments" style="pointer-events: none">Listener function arguments</a> to explore other options for more advanced interactions.</div>
</div>

Here’s what happens when you run the bot and type `/run_jenkins_pipeline` in Slack:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788422626/b3a510fd-db83-4329-8c37-a7ee78c5e685.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788422626/b3a510fd-db83-4329-8c37-a7ee78c5e685.gif)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">For simplicity, we’ve hardcoded the pipeline names in this example. However, you can leverage Slack's advanced <a target="_blank" rel="noopener noreferrer nofollow" href="https://api.slack.com/reference/block-kit/block-elements#select" style="pointer-events: none">Select controls</a> to implement paging, filtering, and dynamic loading from Jenkins. To explore Jenkins API endpoints and retrieve specific data, append <code>/api</code> to any Jenkins page URL (e.g., <a target="_blank" rel="noopener noreferrer nofollow" href="http://jenkins-server/job-name/api" style="pointer-events: none"><code>http://jenkins-server/job-name/api</code></a>).</div>
</div>

With the form now displaying in response to the Slack command, we can move to the next step: teaching the bot how to execute the Jenkins pipeline when the form is submitted.

# Running a Parameterless Pipeline

We’ve added the **Run** and **Cancel** buttons to the form, but they aren’t functional yet. Let’s fix that by using [action handlers](https://slack.dev/bolt-js/concepts/action-respond) to trigger the desired behavior when these buttons are clicked.

The **Cancel** button is straightforward to implement. Its handler simply responds with a message without taking further action. Here's how it works:

```typescript
app.action(ActionKeys.DISPLAY_CANCELLATION_MESSAGE, async ({ack, respond}) => {
    await ack();
    await respond("Ok, I've canceled the operation. See you soon.");
});
```

The **Run** button is where the magic happens. When a user clicks **Run**, we need to do the following:

1. Call `ack()` to notify Slack that we received the action.
    
2. Extract the selected pipeline name from `body.state.values`.
    
3. Start the specified Jenkins Pipeline and return a link to the running job or an error if something goes wrong.
    

Here’s the handler for the **Run** button:

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

The `JenkinsAPI.startJob` function is responsible for triggering the pipeline in Jenkins. It sends a **POST** request to the Jenkins `/build` API, which returns a **queue URL** for the job in the `location` header. After a brief delay, we send a request to the queue URL to retrieve the **running job URL** from the `queuedJobInfo.executable.url` property:

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

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The <code>setTimeout</code> is necessary to allow Jenkins time to move the job from the queue to execution. While it’s a bit of a hack caused by the <a target="_blank" rel="noopener noreferrer nofollow" href="https://www.pluralsight.com/tech-blog/forms-of-temporal-coupling/" style="pointer-events: none">temporal coupling</a>, it’s unavoidable here due to how Jenkins handles job queuing.</div>
</div>

In some cases, Jenkins may be unable to start the job (e.g., no available build agent or [concurrent job execution](https://www.jenkins.io/doc/book/pipeline/syntax/#options) is disabled). When this happens, we return an error message to Slack. The `queuedJobInfo.why` property contains the reason for the failure and is displayed to the user.

So, let's see how it looks in action:

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789052807/72804be2-a5fc-44e1-889b-8a0328c8d5a4.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725789052807/72804be2-a5fc-44e1-889b-8a0328c8d5a4.gif)

The demo shows a user receives a notification when the pipeline is completed. This is achieved using the [Jenkins Slack Notification plugin](https://www.jenkins.io/doc/pipeline/steps/slack/), which allows us to notify users directly in Slack when the job finishes.

The plugin allows to map the build starter’s email to their Slack user ID and send the notification:

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

We also use the `botUser` and `tokenCredentialId` parameters to ensure the bot sends the message, creating a seamless experience where all messages appear in the same conversation.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">You can send the <code>/run_jenkins_pipeline</code> command in any Slack channel or private message, and the bot will respond with <a target="_blank" rel="noopener noreferrer nofollow" href="https://api.slack.com/surfaces/messages#ephemeral" style="pointer-events: none">ephemeral messages</a> that only you can see, ensuring minimal disruption to others. Meanwhile, Jenkins notifications will be sent directly to you via the bot chat.</div>
</div>

# Running Pipelines From Specific User

Currently, we use a single token to interact with the Jenkins API, which means that if our bot is shared with the entire team, all pipelines will be executed under the same user account. This creates a problem: notifications and messages about job statuses will only go to the user who provided the token rather than the individual who triggered the pipeline.

To resolve this, each user should provide their own Jenkins API token. This allows users to run pipelines under their accounts and receive personalized notifications. We can achieve this by using [Slack middleware](https://slack.dev/bolt-js/concepts/listener-middleware) that will:

* **Checks for Saved Credentials:** If the user has already provided their Jenkins credentials, we initialize the Jenkins API instance and [attach it to the context](https://tools.slack.dev/bolt-js/concepts/context/).
    
* **Prompt for Credentials:** If no credentials are found, we open a [modal form](https://slack.dev/bolt-js/concepts/creating-modals) that asks the user to input their Jenkins API token.
    

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788103293/fb003f32-54d4-4613-a086-44019832ba21.gif align="center")](https://cdn.hashnode.com/res/hashnode/image/upload/v1725788103293/fb003f32-54d4-4613-a086-44019832ba21.gif)

Below is the implementation of the middleware:

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

Also, we need to attach the middleware to all listeners that interact with Jenkins. Here’s how you register it with Slack commands and actions:

```typescript
app.command("/run_jenkins_pipeline", constructJenkinsAPI, async ({ack, respond}) => {
...
});

app.action(ActionKeys.RUN_JENKINS_PIPELINE, constructJenkinsAPI, async ({context, body, ack, respond}) => {
...
});
```

The middleware will ensure every user has valid credentials before proceeding with the command or action.

The next piece is implementing the [Slack view listener](https://slack.dev/bolt-js/concepts/view-submissions) called when a user submits the token form. here, we need to save user credentials and confirm the process was successful:

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
<div data-node-type="callout-text">This in-memory storage is just for demonstration purposes. In a production environment, you should use secure storage like <a target="_new" rel="noopener" href="https://www.vaultproject.io/" style="pointer-events: none">HashiCorp Vault</a><a target="_blank" rel="noopener noreferrer nofollow" href="https://www.vaultproject.io/" style="pointer-events: none"> or another sec</a>ret management tool to store credentials safely. Also, implementing secret rotation policy for enhanced security is a good idea.</div>
</div>

With this setup, each team member can now run Jenkins pipelines using their own credentials.

# Running a Parameterized Pipeline

Now that we've handled simple, parameterless pipelines, let’s modify our bot to support running parameterized pipelines as well.

In the updated handler for the `/run_jenkins_pipeline` command, we first check whether the pipeline requires parameters. If it does, we present the user with a form to input those parameters. If not, we simply run the pipeline as before:

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

The key part here is calling the `getJobParameters` method from our Jenkins API class to retrieve the pipeline’s parameter definitions:

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

If no parameters are needed, we run the pipeline immediately by calling the `runParameterlessPipeline` function. It runs parameterless pipeline the same way we implemented previously:

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

For parameterized pipelines, we display a parameters form to the user. Here’s the `buildPipelineParametersForm` function that builds that form:

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

The `mapJenkinsParametersToSlackControls` function maps each Jenkins parameter type to the appropriate Slack UI control. While the [complete implementation](https://github.com/fshchudlo/jenkins-slack-connector/blob/main/actions/helpers/mapJenkinsParametersToSlackControls.ts) is more complex, here’s a simplified version that handles common parameter types:

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
<div data-node-type="callout-text">This switch case is a classic example of violating the <a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle" style="pointer-events: none">Open-Closed Principle</a>. If you plan to handle many control types, consider refactoring with the&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Strategy_pattern" style="pointer-events: none">strategy</a>&nbsp;or&nbsp;<a target="_blank" rel="noopener noreferrer nofollow" href="https://en.wikipedia.org/wiki/Command_pattern" style="pointer-events: none">command</a>&nbsp;pattern for a more scalable solution.</div>
</div>

The last step is implementing another action handler that triggers the pipeline with the provided parameters once the user submits the form. The submitted values are collected with the `getSlackFormValues` function and passed to the `jenkinsAPI.startJob` method:

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

Now, with the parameterized pipeline support in place, we can:

1. Run simple, parameterless pipelines.
    
2. Receive Slack forms for parameterized pipelines, fill them out, and trigger the pipeline with their custom input.
    

Next, we’ll explore a more complex use case for handling interactive pipelines.

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

Integrating Slack with your development tools can elevate your workflow by enabling seamless communication, automation, and collaboration. The true power of such an integration is its ability to connect multiple systems and surface relevant information when needed. Imagine starting your workday with a Slack bot that does the following:

* It lists tasks you've recently closed and suggests follow-up actions like updating documentation or notifying relevant stakeholders.
    
* Displays pull requests you’ve participated in that are awaiting review or haven’t been merged yet, ensuring nothing slips through the cracks.
    
* Shows your next task, along with its recent updates or comments, so you're always in sync with ongoing discussions.
    
* Lists branches you've worked on that lack pull requests, helping you identify abandoned work or unfinished experiments.
    
* Presents recent findings from automated security scans, ensuring security issues are addressed quickly.
    
* Highlights unresolved threads from Slack channels that need your or your team's attention, streamlining internal communication.
    
* If the number of unresolved support tickets spikes, the bot could offer you to help your on-call teammate.
    

By automating these workflows and integrating them into Slack, you can create a cohesive environment tailored to your team’s needs, improving productivity and collaboration.

I hope this guide has equipped you with the tools and knowledge to build your Slack-powered development environment, streamlining your daily operations and enhancing your team's effectiveness.

---

Life is so beautiful,

Fedor