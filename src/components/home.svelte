<script lang="ts">
    import { Link } from "svelte-routing";
    import "bootstrap/dist/css/bootstrap.min.css";

    let initialScale: number;
    let displayBlurryBar: boolean = true; // Variable to control the display of blurry-bar

    function navigateToNew(): void {
        window.location.href = "/new"; // Basic navigation without svelte-routing.
    }

    // Function to get the current zoom scale
    function getZoomScale(): number {
        return window.outerWidth / window.innerWidth;
    }

    // Initial scale
    initialScale = getZoomScale();

    // Event listener for the resize event
    window.addEventListener('resize', function() {
        const currentScale: number = getZoomScale();
        
        // Check if zoomed
        if (currentScale !== initialScale) {
            displayBlurryBar = false;
        } else {
            displayBlurryBar = true;
        }
    });

</script>

<div class="centered-content">
    <img src="/logotarif.png" alt="Logotarif" class="centered-logo" />
    {#if displayBlurryBar}
        <div class="blurry-bar" />
    {/if}
    <span class="textfront">
        Don't know what to eat? Select the ingredients at your house and let AI
        generate the perfect recipe right now.
    </span>
    <Link to="/new">
    <button class="btn btn-warning" on:click={navigateToNew}>
        Try it now🔎
    </button>
    </Link>
</div>

<style>
    /* For webkit browsers */
::-webkit-scrollbar {
    width: 0;  /* Remove scrollbar space */
    background: transparent;  /* Optional: make scrollbar invisible */
}

/* For other browsers */
html {
    scrollbar-width: none; /* Firefox */
    -ms-overflow-style: none;  /* IE and Edge */
}
    body {
        margin: 0;
        padding: 0;
    }
    .centered-content {
        display: flex;
        padding-left: 50%;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        overflow: visible;
        height: 100vh;
        font-size: 1.5rem;
        font-weight: bolder;
        color: #eb9d9e;
        position: relative; /* Necessary for absolute positioning of pseudo-element */
        padding: 30px 0; /* Some padding to give space for the blurred bar */
        left: auto;
    }

    .blurry-bar {
        content: ""; /* Required for pseudo-elements */
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0; /* Makes the pseudo-element cover the full width and height of its parent */
        width: 100vw;
        height: 10vh;
        top: 40vh;
        background: rgba(
            255,
            255,
            255,
            0.1
        ); /* White background with 70% opacity for the blur effect */
        backdrop-filter: blur(10px);
        z-index: 0; /* Position it behind the text */
    }

    button {
        padding: 10px 20px;
        position: relative;
        left: 85%;
        font-size: 1.2rem;
        cursor: pointer;
        color: white;
        border: none;
        border-radius: 5px;
        margin-top: 20px;
        transition: background-color 0.3s;
        box-shadow: #ff0000;
        z-index: 1; /* Ensures the button appears above the blurred bar */
    }

    button:hover {
        background-color: #ff0000;
    }

    .textfront {
        z-index: 1; /* Ensures the text appears above the blurred bar */
        position: relative;
        left: 10%;
    }

    .centered-logo {
        position: absolute;
        top: 10%;
        display: block; 
        margin: 0 auto; 
        transform: scale(0.7);
        padding-left: 30%;
    }
</style>
