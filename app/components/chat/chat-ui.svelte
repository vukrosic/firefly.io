<svelte:options tag="chat-ui" />
<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    let message = '';
    const conversation = writable([]);
    const YOUR_API_KEY = "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";

    const curriculum = [ "Introduction", "Variables and Printing", "Data Types", "Comments", "Console Input", "Arithmetic Operators", "Type Conversions", "Conditions", "Compound Conditions",
        "Conditionals",
        "Strings",
        "Lists",
        "Tuples",
        "Dictionaries",
        "Sets",
        "For Loops",
        "While Loops",
        "Slices",
        "Functions",
        "Scope",
        "Mutability",
        "Exceptions",
        "Math",
        "Sorting",
        "Misc. Python Syntax"
    ];

    let userXP = 0;

    conversation.set([
        {
            role: 'system',
            content: 'You are a Coding Teacher AI teaching me Sorting in python. Use markdown syntax for code. Include a print statement and language name. Explain the code and provide a question or task related to it. Manage XP system: assign 10-20 XP in brackets (XP) for learning and task completion. Partially correct answers get partial XP. User may specify programming language. If not, choose a suitable language. After each response, evaluate the response and add the XP. Always end with "Your XP: XP". One concept per message, no greetings or extra words. Await user\'s direction to proceed.',
        },
        {
            role: 'assistant',
            content: '**Curriculum:**\n\nPython Lessons\n1 - Introduction\n2 - Variables and Printing\n3 - Data Types\n4 - Comments\n5 - Console Input\n6 - Arithmetic Operators\n7 - Type Conversions\n8 - Conditions\n9 - Compound Conditions\n10 - Conditionals\n11 - Strings\n12 - Lists\n13 - Tuples\n14 - Dictionaries\n15 - Sets\n16 - For Loops\n17 - While Loops\n18 - Slices\n19 - Functions\n20 - Scope\n21 - Mutability\n22 - Exceptions\n23 - Math\n24 - Sorting\n25 - Misc. Python Syntax',
        },
    ]);

    function getNextLesson(index) {
        const sentence = 'You are a Coding Teacher AI teaching me _____ in Python. Add XP to the user if he answers your questions correctly. Always end with "Your XP: XP".';
        const topic = curriculum[index];
        return sentence.replace('_____', topic);
    }

    function giveXP(points) {
        userXP += points;
        return `Your XP: ${userXP}`;
    }

    function parseXPFromResponse(response) {
        const xpRegex = /Your XP: (\d+)/;
        const match = response.match(xpRegex);
        return match ? parseInt(match[1], 10) : 0;
    }

    // Create a new function to handle sending custom prompts to GPT
    const sendPrompt = async (prompt) => {
        if (!prompt.trim()) return;
        message = prompt;
        await callOpenAI();
    };
    // Modify the callOpenAI function to accept a custom prompt
    const callOpenAI = async (customPrompt = null) => {
        const prompt = customPrompt || message;
        if (!prompt.trim()) return;
        if (!message.trim()) return;

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


    // Add click event listeners to the buttons
    const explainMore = () => {
        const lastResponse = $conversation[$conversation.length - 1].content;
        sendPrompt(`Explain the last response in more detail: ${lastResponse}`);
    };

    const teachNext = () => {
        sendPrompt('Teach me the next thing.');
    };
</script>

<main>
    <h1>OpenAI API Call</h1>
    <label for="message">Message:</label>
    <input
            type="text"
            id="message"
            bind:value="{message}"
            placeholder="Type your message here"
    />
    <button on:click="{callOpenAI}">Send Message</button>
    <h2>Conversation:</h2>
    <ul>
        {#each $conversation as { role, content }}
            <li><strong>{role}:</strong> {content}</li>
        {/each}
    </ul>
    <button on:click="{explainMore}">Explain in more detail</button>
    <button on:click="{teachNext}">Teach me the next thing</button>
</main>




<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    label,
    input,
    button,
    h2 {
        margin-bottom: 8px;
    }
</style>