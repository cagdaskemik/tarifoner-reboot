<script>
	import { Router, Route } from "svelte-routing";
	import Welcome from "./components/home.svelte";
	import Main from "./components/new.svelte"; // The new page component you want to navigate to.
	import { inject } from "@vercel/analytics";

	inject();
	
</script>

<Router>
	<Route path="/" component={Welcome} />
	<Route path="/new" component={Main} />
</Router>
