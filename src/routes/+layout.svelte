<svelte:head>
  <link rel="stylesheet" href="/css/layout.css" />
</svelte:head>


<header>
    <img src="/img/logo.png" alt="Logo" class="logo" />
    <h1>Vibe & Dine Skopje</h1>
</header>

<nav>
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/map">Explore</a></li>
        <li><a href="/profile">Profile</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>


    </ul>
</nav>

<main>
    <slot></slot>
</main>

<footer class="footer">
    <div class="footer-container">
        <div class="footer-logo">
            <img src="/img/logo.png" alt="Vibe & Dine Skopje Logo">
            <h3>Vibe & Dine Skopje</h3>
        </div>

        <div class="footer-links">
            <a href="/">Home</a>
            <a href="/map">Explore</a>
            <a href="/profile">Profile</a>
            <a href="/about">About</a>
            <a href="/contact">Contact</a>
        </div>

        <div class="footer-socials">
            <a href="#" target="_blank"><img src="/icons/facebook.png" alt="Facebook"></a>
            <a href="#" target="_blank"><img src="/icons/instagram.png" alt="Instagram"></a>
            <a href="#" target="_blank"><img src="/icons/twitter.png" alt="Twitter"></a>
        </div>
    </div>

    <div class="footer-bottom">
        <p>&copy; {new Date().getFullYear()} Vibe & Dine Skopje. All rights reserved.</p>
    </div>
</footer>

<!-- <style>
    @import '/css/layout.css';
</style> -->