<svelte:options tag="chat-ui" />
<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { marked } from 'marked';
    // import Prism from 'prismjs';
    // import "prismjs/themes/prism.css";
    // import 'prismjs/components/prism-python';
    // import 'prismjs/components/prism-markdown';
    
    // let codeElement;
    // onMount(() => {
    //     Prism.highlightElement(codeElement);
    // });


    // $: if (codeElement) {
    // Prism.highlightElement(codeElement);
    // }

 


    let message = '';
    const conversation = writable([]);
    const YOUR_API_KEY = "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";


    let userXP = 0;

    conversation.set([
        {
            role: 'system',
            content: "Ask technical Python few-line code snippet questions and offer a), b) and c) as given answers to choose from. Make it hard. Respond in markdown. Only one question at a time. After user's answer just reply with \"Correct. Next questions:\" or  \"Incorrect. Next questions:\" and continue with the next question. Then ask the next question without any further explanation.",
        },
        {
            role: 'assistant',
            content: "Question 1: What is the output of the following Python code snippet?\n\n```python\ndef foo(n):\n    return n * 2\n\nresult = [foo(x) for x in range(3)]\nprint(result)\n```\n\na) `[0, 2, 4]`\nb) `6`\nc) `TypeError`",
        },
        {
            role: 'user',
            content: "a"
        },
        {
            role: 'assistant',
            content: "Correct. Next question:\n\nQuestion 2: What is the output of the following Python code snippet?\n\n```python\ndef bar(a, b):\n    return a + b, a * b\n\nx, y = bar(2, 3)\nprint(x, y)\n```\n\na) `5 6`\nb) `(5, 6)`\nc) `TypeError`",
        },
        {
            role: 'user',
            content: "a"
        },
        {
            role: 'assistant',
            content: "Correct. Next question:\n\nQuestion 3: What is the output of the following Python code snippet?\n\n```python\nx = 5\ny = 2\n\nresult = x / y\nprint(result)\n```\n\na) `2.5`\nb) `2`\nc) `2.0`",
        },
        {
            role: 'user',
            content: "b"
        }
    ]);

    // function getNextLesson(index) {
    //     const sentence = 'You are a Coding Teacher AI teaching me _____ in Python. Add XP to the user if he answers your questions correctly. Always end with "Your XP: XP".';
    //     const topic = curriculum[index];
    //     return sentence.replace('_____', topic);
    // }

    // function giveXP(points) {
    //     userXP += points;
    //     return `Your XP: ${userXP}`;
    // }

    // function parseXPFromResponse(response) {
    //     const xpRegex = /Your XP: (\d+)/;
    //     const match = response.match(xpRegex);
    //     return match ? parseInt(match[1], 10) : 0;
    // }

    // function clearConversation() {
    //     conversation.set([
    //         {
    //         role: 'system',
    //         content: 'You are a Coding Teacher AI teaching me Sorting in python. Use markdown syntax for code. Include a print statement and language name. Explain the code and provide a question or task related to it. Manage XP system: assign 10-20 XP in brackets (XP) for learning and task completion. Partially correct answers get partial XP. User may specify programming language. If not, choose a suitable language. After each response, evaluate the response and add the XP. Always end with "Your XP: XP". One concept per message, no greetings or extra words. Await user\'s direction to proceed.',
    //         }
    //     ]);
    // }

    // Create a new function to handle sending custom prompts to GPT
    const sendPrompt = async (prompt) => {
        if (!prompt.trim()) return;
        message = prompt;
        await callOpenAI();
    };
    // Modify the callOpenAI function to accept a custom prompt
    const callOpenAI = async (customPrompt = null) => {
        //const prompt = customPrompt || message;
        // if (!prompt.trim()) return;
        // if (!message.trim()) return;

        conversation.update(conv => [...conv, { role: 'user', content: message }]);

        const requestBody = {
            model: 'gpt-3.5-turbo',
            messages: $conversation,
            stream: true,
        };

        const response = await fetch('https://api.openai.com/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${YOUR_API_KEY}`,
            },
            body: JSON.stringify(requestBody),
        });
        message = '';
        if (response.status === 200) {
            const reader = response.body.getReader();
            let decoder = new TextDecoder();
            let partialResponse = '';

            conversation.update(conv => [...conv, { role: 'assistant', content: '' }]);
            let assistantMessageIndex = $conversation.length - 1;

            while (true) {
                const { done, value } = await reader.read();
                if (done) break;
                partialResponse = decoder.decode(value);
                const lines = partialResponse.split('\n');

                for (const line of lines) {
                    if (line === 'data: [DONE]') break;

                    if (line.startsWith('data: ')) {
                        const parsedData = JSON.parse(line.slice(6));
                        const content = parsedData.choices[0]?.delta?.content || '';

                        if (content) {
                            conversation.update(conv => {
                                const updatedConv = [...conv];
                                updatedConv[assistantMessageIndex].content += content;
                                return updatedConv;
                            });
                        }
                    }
                }
            }
        } else {
            conversation.update(conv => [...conv, { role: 'assistant', content: 'Error while fetching response from OpenAI API.' }]);
        }

        message = '';
    };
    // const html = marked.parse('Question 1: What is the output of the following Python code snippet?\n\n```python\ndef foo(n):\n    return n * 2\n\nresult = [foo(x) for x in range(3)]\nprint(result)\n```\n\na) `[0, 2, 4]`\nb) `6`\nc) `TypeError`');

    // console.log(html);
    $: html = marked($conversation.map(msg => `**${msg.role}:** ${msg.content}`).join('\n\n'));


    // // Add click event listeners to the buttons
    // const explainMore = () => {
    //     sendPrompt(`Explain the last response in more detail.`);
    // };

    // const teachNext = () => {
    //     sendPrompt('Teach me the next thing.');
    // };
</script>
    <main>
    <h1>Personal Teacher GPT</h1>
    <ul>
        {@html html}
    </ul>
    <!-- <ul>
        {#each $conversation as { role, content }}
            {#if role !== 'system'}
                <li><strong>{role}:</strong> {content}</li>
            {/if} 
        {/each}
    </ul> -->
    <div>
        <button class="material-button" on:click="{() => sendPrompt('a')}">a</button>
        <button class="material-button" on:click="{() => sendPrompt('b')}">b</button>
        <button class="material-button" on:click="{() => sendPrompt('c')}">c</button>
    </div>
    <div class="message-area">
        <label for="message">Message:</label>
        <input
                type="text"
                id="message"
                bind:value="{message}"
                placeholder="Type your message here"
        />
    </div>
    <div class="button-group">
        <button class="material-button" on:click="{callOpenAI}">Send Message</button>
        <!-- <button class="material-button" on:click="{explainMore}">Explain in more detail</button>
        <button class="material-button" on:click="{teachNext}">Teach me the next thing</button> -->
        <!-- <button class="material-button" on:click="{clearConversation}">Clear Conversation</button> -->
    </div>
</main>


<svelte:head>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.22.0/themes/prism.min.css" rel="stylesheet" />
</svelte:head>



<!-- <style>
    * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
        font-family: 'Roboto', sans-serif;
    }

    /* body {
        background-color: #f2f2f2;
    } */

    main {
        max-width: 800px;
        margin: 50px auto;
        background-color: #ffffff;
        border-radius: 10px;
        box-shadow: 0px 3px 10px rgba(0, 0, 0, 0.1);
        padding: 20px;
    }

    h1, h2 {
        color: #333;
        padding: 8px;
        text-align: center;
    }

    ul {
        max-height: 400px;
        overflow-y: auto;
        padding: 16px;
        border: 1px solid #e0e0e0;
        margin-bottom: 20px;
        list-style-type: none;
        border-radius: 5px;
    }

    li {
        padding: 10px;
        border-bottom: 1px solid #e0e0e0;
        line-height: 30px;
    }

    li:last-child {
        border-bottom: none;
    }

    strong {
        font-weight: 500;
        color: #4a4a4a;
    }

    label {
        display: block;
        margin-bottom: 10px;
        font-weight: 500;
        color: #4a4a4a;
    }

    input {
        width: 100%;
        padding: 10px 15px;
        font-size: 16px;
        border: 1px solid #e0e0e0;
        border-radius: 5px;
    }

    button {
        cursor: pointer;
        padding: 9px 14px;
        font-size: 16px;
        border: none;
        border-radius: 5px;
        margin-right: 10px;
    }

    button:last-child {
        margin-right: 0;
    }

    button:hover {
        opacity: 0.85;
    }

    /* .primary {
        background-color: #488ffc;
        color: #333;
    }

    .secondary {
        background-color: #eeeeee;
        color: #333;
    } */
</style> -->