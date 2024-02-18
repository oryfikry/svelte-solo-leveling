<script>
	import { onMount, onDestroy } from "svelte";
	import { writable } from "svelte/store";
  
	let imageData = [];
	let currentPage = 1;
	const pageParam = writable(currentPage);
  
	const fetchImageData = async (page) => {
	  try {
		const response = await fetch(`images/output-solo-leveling-${page}.json`);
		const data = await response.json();
		imageData = data.urlData;
	  } catch (error) {
		console.error("Error fetching data:", error);
	  }
	};
  
	const nextPage = () => {
	  pageParam.update((value) => {
		const newPage = value + 1;
		fetchImageData(newPage);
		window.history.pushState({}, "", `?page=${newPage}`);
		scrollToTop();
		return newPage;
	  });
	};
  
	const prevPage = () => {
	  if (currentPage > 1) {
		pageParam.update((value) => {
		  const newPage = value - 1;
		  fetchImageData(newPage);
		  window.history.pushState({}, "", `?page=${newPage}`);
		  scrollToTop();
		  return newPage;
		});
	  }
	};
  
	const scrollToTop = () => {
	  window.scrollTo({ top: 0, behavior: "smooth" });
	};
  
	onMount(() => {
	  const paramsSubscription = pageParam.subscribe((value) => {
		currentPage = value;
		fetchImageData(value);
	  });
  
	  window.addEventListener("popstate", () => {
		pageParam.set(parseInt(new URLSearchParams(window.location.search).get("page") || currentPage, 10));
	  });
  
	  onDestroy(() => {
		paramsSubscription();
		window.removeEventListener("popstate", () => {
		  pageParam.set(parseInt(new URLSearchParams(window.location.search).get("page") || currentPage, 10));
		});
	  });
	});
  </script>
  
  <style>
	img {
	  max-width: 100%;
	  height: auto;
	  display: block;
	  margin-bottom: 20px;
	}
  
	main {
	  padding: 20px;
	}
  
	.floating-buttons {
	  position: fixed;
	  bottom: 20px;
	  left: 50%;
	  transform: translateX(-50%);
	  display: flex;
	  justify-content: space-between;
	  width: 200px;
	}
  
	button {
	  padding: 10px;
	  cursor: pointer;
	  border: none;
	  background-color: #4caf50;
	  color: white;
	  border-radius: 5px;
	  font-size: 14px;
	  transition: background-color 0.3s;
	}
  
	button:hover {
	  background-color: #45a049;
	}
  </style>
  
  <main>
	{#each imageData as url (url)}
	  <img src={url} alt={`Image`} />
	{/each}
  
	<div class="floating-buttons">
	  <button on:click={prevPage} disabled={currentPage === 1}>
		Previous Page
	  </button>
	  <button on:click={nextPage}>
		Next Page
	  </button>
	</div>
  </main>
  