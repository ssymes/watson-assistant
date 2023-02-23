---

copyright:
  years: 2022, 2023
lastupdated: "2023-02-16"

subcollection: watson-assistant

---

{{site.data.keyword.attribute-definition-list}}

# Tutorial: Displaying a pre-chat or post-chat form
{: #web-chat-develop-pre-chat}

This tutorial shows how you can display pre-chat form before the web chat opens, or a post-chat form that opens after the web chat closes.
{: shortdesc}

For a complete, working version of the example described in this tutorial, see [Pre and post-chat forms for {{site.data.keyword.conversationshort}} web chat](https://github.com/watson-developer-cloud/assistant-toolkit/tree/master/integrations/webchat/examples/pre-post-chat-forms){: external}.
{: note}

If you want to gather information from your customers before starting a chat session, you can display a pre-chat form before opening the web chat. Similarly, you might want to display a form after the web chat closes (for example, a customer satisfaction survey). You can use the same approach for either situation.

When the web chat is opened or closed, it fires an [event](https://web-chat.global.assistant.watson.cloud.ibm.com/docs.html?to=api-events){: external} that you can subscribe to. In your event handler, you can use the [custom panels](https://web-chat.global.assistant.watson.cloud.ibm.com/docs.html?to=api-render#custompanel){: external} feature to open a panel with custom content.

By returning a promise that is resolved when the custom panel closes, you can pause the process of opening or closing the web chat until after the customer completes the form.

This example shows how to create a pre-chat form. To create a post-chat form, follow the same steps, but subscribe to the `window:pre:close` event instead of the `window:open` event.
{: tip}

To display a pre-chat form, follow these steps:

1. Create a handler for the [`window:open`](https://web-chat.global.assistant.watson.cloud.ibm.com/docs.html?to=api-events#windowpreopen){: external} event, which is fired when the web chat opens. This handler uses the [`customPanels.getPanel()`](/docs/watson-assistant?topic=watson-assistant-topicid){: external} instance method to open a custom panel that will contain the pre-chat form.

    Your handler should return a promise that is resolved when the custom panel is closed. This prevents the web chat window from opening until after the pre-chat form is completed.
    {: tip}

    ```javascript
    function windowOpenHandler(event, instance) {
      return new Promise((resolve) => {
        // Save a reference to the resolve function so we can resolve 
        // this promise later.
        promiseResolve = resolve;
        createOpenPanel(event, instance);
    
        const customPanel = instance.customPanels.getPanel();
        customPanel.open({ hidePanelHeader: true,
                           disableAnimation: true });
      });
    }
    ```
    {: codeblock}

1. In your `onLoad` event handler, use the [`on()`](https://web-chat.global.assistant.watson.cloud.ibm.com/docs.html?to=api-instance-methods#on){: external} instance method to subscribe to the `window:open` event, registering the handler as the callback.

    ```javascript
    instance.on({ type: 'window:open', handler: windowOpenHandler });
    ```
    {: codeblock}

1. Create a function that creates the pre-chat form you want to show inside the custom panel. Make sure you resolve the promise when the user closes the panel.

    ```javascript
    function createOpenPanel(event, instance) {
      const customPanel = instance.customPanels.getPanel();
      const { hostElement } = customPanel;
  
      hostElement.innerHTML = `
        <div class="PreChatForm">
          ... // Content of pre-chat form
        </div>
      `;

      const okButton = hostElement.querySelector('#PreChatForm__OkButton')
      okButton.addEventListener('click', () => {
        customPanel.close();
        promiseResolve();
      });
    }
    ```
    {: codeblock}

A [React version](https://github.com/watson-developer-cloud/assistant-toolkit/tree/master/integrations/webchat/examples/pre-post-chat-forms/client/react){: external} of this example is also available.
{: tip}

For complete working code, see [Pre and post-chat forms for {{site.data.keyword.conversationshort}} web chat](https://github.com/watson-developer-cloud/assistant-toolkit/tree/master/integrations/webchat/examples/pre-post-chat-forms){: external} example.
