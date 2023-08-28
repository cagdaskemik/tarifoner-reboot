<!-- src/App.svelte -->

<script lang="ts">
    import { onValue } from "firebase/database";
    import { writable } from "svelte/store";
    import type { Writable } from "svelte/store";
    import "bootstrap/dist/css/bootstrap.min.css";
    import { initializeApp } from "firebase/app";
    import { getDatabase, ref } from "firebase/database";

    type IngredientType = {
        name: string;
        type: string;
        active: boolean;
    };
    type ImageUrlsType = {
        [key: string]: string;
    };

    interface FirebaseConfig {
        apiKey: string;
        authDomain: string;
        databaseURL: string;
        projectId: string;
        storageBucket: string;
        messagingSenderId: string;
        appId: string;
    }

    const firebaseConfig: FirebaseConfig = {
        apiKey: import.meta.env.VITE_API_KEY,
        authDomain: import.meta.env.VITE_AUTH_DOMAIN,
        databaseURL: import.meta.env.VITE_DATABASE_URL,
        projectId: import.meta.env.VITE_PROJECT_ID,
        storageBucket: import.meta.env.VITE_STORAGE_BUCKET,
        messagingSenderId: import.meta.env.VITE_MESSAGING_SENDER_ID,
        appId: import.meta.env.VITE_APP_ID,
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    const svgContent = `
        <svg id='svg' xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-database-fill-x" viewBox="0 0 16 16">
			<path d="M8 1c-1.573 0-3.022.289-4.096.777C2.875 2.245 2 2.993 2 4s.875 1.755 1.904 2.223C4.978 6.711 6.427 7 8 7s3.022-.289 4.096-.777C13.125 5.755 14 5.007 14 4s-.875-1.755-1.904-2.223C11.022 1.289 9.573 1 8 1Z"/>
			<path d="M2 7v-.839c.457.432 1.004.751 1.49.972C4.722 7.693 6.318 8 8 8s3.278-.307 4.51-.867c.486-.22 1.033-.54 1.49-.972V7c0 .424-.155.802-.411 1.133a4.51 4.51 0 0 0-4.815 1.843A12.31 12.31 0 0 1 8 10c-1.573 0-3.022-.289-4.096-.777C2.875 8.755 2 8.007 2 7Zm6.257 3.998L8 11c-1.682 0-3.278-.307-4.51-.867-.486-.22-1.033-.54-1.49-.972V10c0 1.007.875 1.755 1.904 2.223C4.978 12.711 6.427 13 8 13h.027a4.552 4.552 0 0 1 .23-2.002Zm-.002 3L8 14c-1.682 0-3.278-.307-4.51-.867-.486-.22-1.033-.54-1.49-.972V13c0 1.007.875 1.755 1.904 2.223C4.978 15.711 6.427 16 8 16c.536 0 1.058-.034 1.555-.097a4.507 4.507 0 0 1-1.3-1.905Z"/>
			<path d="M12.5 16a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7Zm-.646-4.854.646.647.646-.647a.5.5 0 0 1 .708.708l-.647.646.647.646a.5.5 0 0 1-.708.708l-.646-.647-.646.647a.5.5 0 0 1-.708-.708l.647-.646-.647-.646a.5.5 0 0 1 .708-.708Z"/>
        </svg>
    `;
    const svgDataURL =
        "data:image/svg+xml;utf8," + encodeURIComponent(svgContent);

    const ingredientsRef = ref(db, "items");
    let ingredients: Array<{ name: string; type: string; active: boolean }> =
        [];
    let searchQuery: string;
    let selectedIngredients: string[] = [];
    let pressedButtons: string[] = [];
    let filteredIngredients: Writable<
        Array<{ name: string; type: string; active: boolean }>
    > = writable([]);
    const imageUrls: Writable<ImageUrlsType> = writable({});
    let filterCategories = [
        "Spices & Seasonings",
        "Fruits & Vegetables",
        "Meats & Seafoods",
        "Dairy & Alternatives",
        "Grains & Legumes",
        "Bakery & Sweets",
        "Oils & Condiments",
        "Others",
    ];

    const ITEMS_PER_PAGE = 16;
    let currentPage = 1;
    let paginatedIngredients: Writable<
        Array<{ name: string; type: string; active: boolean }>
    > = writable([]);
    let selectedCategoriesStore: Writable<string[]> = writable([]);

    let command: string =
        "Ingredients on hand: " +
        selectedIngredients.toString() +
        " Given the above ingredients, please provide a recipe that I can prepare using only these items, including step-by-step instructions.";

    let buttonText = "Copy to Clipboard and Open ChatGPT";

    function filterByCategory(category: string) {
        selectedCategoriesStore.update((selectedCategories) => {
            if (selectedCategories.includes(category)) {
                return selectedCategories.filter((cat) => cat !== category);
            } else {
                return [...selectedCategories, category];
            }
        });

        if ($selectedCategoriesStore.length === 0) {
            filteredIngredients.set(ingredients);
        } else {
            const result = ingredients.filter((ingredient) =>
                $selectedCategoriesStore.includes(ingredient.type)
            );
            filteredIngredients.set(result);
        }
        currentPage = 1;
        paginate();
    }

    function paginate() {
        const startIndex = (currentPage - 1) * ITEMS_PER_PAGE;
        const endIndex = startIndex + ITEMS_PER_PAGE;
        paginatedIngredients.set(
            $filteredIngredients.slice(startIndex, endIndex)
        );
    }
    function nextPage() {
        if (
            currentPage <
            Math.ceil($filteredIngredients.length / ITEMS_PER_PAGE)
        ) {
            currentPage++;
            paginate();
        }
    }

    function prevPage() {
        if (currentPage > 1) {
            currentPage--;
            paginate();
        }
    }

    function search() {
        const result = ingredients.filter((ingredient: { name: string }) =>
            ingredient.name.toLowerCase().includes(searchQuery.toLowerCase())
        );

        filteredIngredients.set(result);
        currentPage = 1;
        paginate();
    }

    function selectIngredient(ingredient: { name: string }) {
        if (selectedIngredients.includes(ingredient.name)) {
            selectedIngredients = selectedIngredients.filter(
                (item) => item !== ingredient.name
            );

            // Also remove from pressedButtons
            pressedButtons = pressedButtons.filter(
                (item) => item !== ingredient.name
            );
            console.log(pressedButtons);
            console.log(pressedButtons.includes(ingredient.name));
        } else {
            selectedIngredients = [...selectedIngredients, ingredient.name];

            // Also add to pressedButtons
            pressedButtons.push(ingredient.name);
            console.log(pressedButtons);
            console.log(pressedButtons.includes(ingredient.name));
        }
    }

    function preloadImage(ingredientName: string) {
        const imageUrl = `https://www.themealdb.com/images/ingredients/${ingredientName}.png`;

        let img = new Image();
        img.src = imageUrl;

        img.onload = function () {
            imageUrls.update((urls: ImageUrlsType) => {
                urls[ingredientName] = imageUrl;
                return urls;
            });
        };
    }

    onValue(ingredientsRef, (snapshot) => {
        ingredients = snapshot.val().map((ingredient: IngredientType) => ({
            ...ingredient,
            active: false,
        }));

        ingredients.forEach((ingredient) => preloadImage(ingredient.name));

        filteredIngredients.set(ingredients); // Only set the filtered ingredients here
        paginate(); // Paginate based on filtered ingredients
    });

    $: isActive = (ingredientName: string) =>
        selectedIngredients.includes(ingredientName);

    let loaded = false;
    let items = []; // Put your array items here

    function popupToggle() {
        loaded = !loaded;
    }

    async function copyToClipboardAndRedirect() {
        // Change the text of the button
        buttonText = "Copied✔️ Redirecting...";

        // Update command right before copying.
        command =
            "Ingredients on hand: " +
            selectedIngredients.join(", ") +
            ". Given the above ingredients, please provide a recipe that I can prepare using only these items, including step-by-step instructions.";

        await navigator.clipboard.writeText(command);
        setTimeout(() => {
            window.open("http://chat.openai.com", "_blank"); // Open in a new window.
            buttonText = "Copy to Clipboard and Open ChatGPT";
        }, 1000);
    }
</script>

<div class="new">
    <div id="survey-popup" class={loaded ? "" : "closed"}>
        <div class="survey-container loading">
            <ul>
                {#each selectedIngredients as ingredientName}
                    <li class="selectedList">{ingredientName}</li>
                {/each}
            </ul>
        </div>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div class="survey-popup-button" on:click={popupToggle}>CHECKLIST:</div>
    </div>

    <nav class="navbar">
        <input
            class="form-control me-2 p-2 m-3"
            bind:value={searchQuery}
            on:input={search}
            placeholder="Search Ingredients..."
        />
        <i class="bi bi-check2-square" />

        <div class="dropdown p-5">
            <button
                class="btn btn-warning dropdown-toggle"
                id="categoryDropdown"
                data-bs-toggle="dropdown"
                aria-expanded="false"
            >
                Filters
            </button>
            <ul class="dropdown-menu" aria-labelledby="categoryDropdown">
                {#each filterCategories as category}
                    <li>
                        <!-- svelte-ignore a11y-click-events-have-key-events -->
                        <!-- svelte-ignore a11y-missing-attribute -->
                        <!-- svelte-ignore a11y-no-static-element-interactions -->
                        <a
                            class="dropdown-item"
                            on:click={() => filterByCategory(category)}
                        >
                            {category}
                            <!-- Display SVG if category is selected -->
                            {#if $selectedCategoriesStore.includes(category)}
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    width="16"
                                    height="16"
                                    fill="currentColor"
                                    class="bi bi-check2-square"
                                    viewBox="0 0 16 16"
                                >
                                    <path
                                        d="M3 14.5A1.5 1.5 0 0 1 1.5 13V3A1.5 1.5 0 0 1 3 1.5h8a.5.5 0 0 1 0 1H3a.5.5 0 0 0-.5.5v10a.5.5 0 0 0 .5.5h10a.5.5 0 0 0 .5-.5V8a.5.5 0 0 1 1 0v5a1.5 1.5 0 0 1-1.5 1.5H3z"
                                    />
                                    <path
                                        d="m8.354 10.354 7-7a.5.5 0 0 0-.708-.708L8 9.293 5.354 6.646a.5.5 0 1 0-.708.708l3 3a.5.5 0 0 0 .708 0z"
                                    />
                                </svg>
                            {/if}
                        </a>
                    </li>
                {/each}
            </ul>
        </div>
    </nav>

    <div class="buttons-container">
        {#each $paginatedIngredients as ingredient}
            <button
                on:click={() => selectIngredient(ingredient)}
                class="card-body gradient-buttons btn btn-danger btn-lg btn3d {isActive(
                    ingredient.name
                )
                    ? 'active'
                    : ''}"
            >
                {#if $imageUrls[ingredient.name]}
                    <img
                        class="img-fluid"
                        src={$imageUrls[ingredient.name]}
                        alt={ingredient.name}
                        width="50"
                        height="50"
                    />
                    <img
                        class="img-shadow"
                        src={$imageUrls[ingredient.name]}
                        alt={ingredient.name}
                        width="50"
                        height="50"
                    />
                {:else}
                    <img
                        class="img-fallback"
                        src={svgDataURL}
                        alt={ingredient.name}
                        width="30"
                        height="30"
                    />
                {/if}

                <span class="name">{ingredient.name}</span>
                <span class="type">{ingredient.type}</span>
            </button>
        {/each}
    </div>

    <!-- Pagination -->
    <!--  -->
    <div class="pagination">
        <button
            class="btn btn-warning"
            on:click={prevPage}
            disabled={currentPage === 1}
        >
            <span class="bold">&lt; Prev</span>
        </button>
        <span class="bold text-warning pt-2 ml-1 mr-1">
            {currentPage} / {Math.ceil(
                $filteredIngredients.length / ITEMS_PER_PAGE
            )}
        </span>

        <button
            class="btn btn-warning"
            on:click={nextPage}
            disabled={currentPage ===
                Math.ceil($filteredIngredients.length / ITEMS_PER_PAGE)}
        >
            <span class="bold">Next &gt;</span>
        </button>
    </div>

    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="redirect btn btn-warning" on:click={copyToClipboardAndRedirect}>
        {buttonText}
    </div>
    <div class="construction">
        ❗Recipe Generator API is under maintenance. You can copy the prompt
        using above button and redirect to ChatGPT.❗
    </div>
</div>
