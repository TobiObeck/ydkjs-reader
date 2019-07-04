<script>
  export let heading = "";
  export let paragraph = "";
</script>

<div class=" md:w-1/2 lg:w-1/3 p-2">
  <section
    class="px-4 py-3 bg-gray-100 rounded-lg shadow-md hover:scale-sm
    hover:shadow-lg trans-ease-out trans-100 trans">
    <h4 class="text-xl mb-1">{heading}</h4>
    <p> {paragraph} </p>
  </section>
</div>
