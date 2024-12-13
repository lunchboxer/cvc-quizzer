<script>
    import { page } from '$app/stores';
    console.log($page.error)
</script>

<div class="error-container">
    <h1>Error {$page.status}</h1>
    <p>{$page.error?.message || 'An unexpected error occurred'}</p>
    <a href="/">Return to Home</a>
</div>

<style>
    .error-container {
        display: flex;
        padding: 2rem;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100svh;
        text-align: center;
        font-family: inherit;
        background-color: #f0f0f0;
        box-sizing: border-box;
    }

    h1 {
        color: #d9534f;
        font-size: 2.5rem;
        margin-bottom: 1rem;
    }

    p {
        color: #666;
        margin-bottom: 1.5rem;
        max-width: 600px;
    }

    a {
        background-color: #3498db;
        color: white;
        padding: 0.5rem 1rem;
        text-decoration: none;
        border-radius: 5px;
        transition: background-color 0.3s;
    }

    a:hover {
        background-color: #2980b9;
    }
</style>
