<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    let message = '';
    let response = writable('');
    let YOUR_API_KEY = "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";

    const callOpenAI = async () => {
        const requestBody = {
            model: 'gpt-3.5-turbo',
            messages: [{ role: 'user', content: message }],
        };

        const responseJson = await fetch('https://api.openai.com/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${YOUR_API_KEY}`,
            },
            body: JSON.stringify(requestBody),
        });

        if (responseJson.status === 200) {
            const data = await responseJson.json();
            response.set(data.choices[0]?.message?.content || 'No response received.');
        } else {
            response.set('Error while fetching response from OpenAI API.');
        }
    };

    onMount(() => {
        callOpenAI();
    });
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
    <h2>Response:</h2>
    <p>{$response}</p>
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