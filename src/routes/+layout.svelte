<script>
    import "../app.css"
    export let children;
</script>

<nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
</nav>

{@render children()}
