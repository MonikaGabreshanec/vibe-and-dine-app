<section class="hero">
    <h2>✨ Skopje’s Ultimate Eats & Beats ✨</h2>
    <p>
        Savor. Sip. Dance. Repeat. From cozy cafés to buzzing bars,
        gourmet restaurants to electrifying nightclubs, Skopje’s food
        and nightlife scene has it all. Whether you're looking for a laid-back drink,
        a fine dining experience, or an unforgettable night out, this city never sleeps.
    </p>
    <a href="/map" class="explore-btn">Start Exploring</a>
</section>
<section class="categories">
    <h3>Discover the Best Spots</h3>
    <div class="category-grid">
        <div class="category-card" on:click={() => startExploring('cafe')}>
            <img src="/icons/cafe.png" alt="Cafes">
            <h4>Cafes</h4>
        </div>
        <div class="category-card" on:click={() => startExploring('pub')}>
            <img src="/icons/pub.png" alt="Pubs">
            <h4>Pubs</h4>
        </div>
        <div class="category-card" on:click={() => startExploring('bar')}>
            <img src="/icons/bar.png" alt="Bars">
            <h4>Bars</h4>
        </div>
        <div class="category-card" on:click={() => startExploring('restaurant')}>
            <img src="/icons/restaurant.png" alt="Restaurants">
            <h4>Restaurants</h4>
        </div>
    </div>
</section>
<section class="recommendations">
    <h3>Recommended Places to Visit</h3>

</section>


<script lang="ts">
    function startExploring(category: string) {
        window.location.href = `/map?category=${category}`; // Navigate to the map with category filter
    }
</script>

<style>
     @import '/css/home.css';
</style>