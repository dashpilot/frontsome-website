<script>
  import { onMount } from 'svelte';
  import { tick } from 'svelte';
  import Navigo from "navigo";

  export let data;
  let router;
  let posts;
  let notfound = false;
  
  let code = `const about = () => {
    console.log('Frontsome is a library and blog')
    console.log('about awesome front-end projects.')
  }`
  
  let mycode = ""


  if(data && data.ssr && data.isCat){      
    posts = data.posts.filter(x=>x.category==data.category)
  }
  if(data && data.ssr && data.isPost){      
    posts = data.posts.filter(x=>x.id==data.id)
  }
  
  onMount(async () => {
    var resp = await fetch(
      cfg.dataPath, {cache: "no-store"}
    );
    data = await resp.json();
    console.log(data)

    router = new Navigo("/");

    router.on("/", async function () {
      console.log("home")
      notfound = false;
      posts = data.posts.filter(x=>x.category=='home')
      
      hydrate('home')
    });
    
    data.categories.forEach(item=>{
      router.on("/"+item.slug, async function () {
      
        notfound = false;
        posts = data.posts.filter(x=>x.category==item.slug)
        
        hydrate(item.slug);
      });
    })
    
    data.posts.forEach(item=>{
      router.on("article/"+slugify(item.title, item.id), async function () {
     
        
        notfound = false;
        posts = data.posts.filter(x=>x.id==item.id)
        
        hydrate(item.category);
      });
    })
    
    router.notFound(() => {
      // this runs if there is no match found
      notfound = true;
    });
  
    
    setTimeout(()=>{
      router.resolve();
    }, 50)
    
   
      // this allows you to update the data externally
      document.body.addEventListener("update", (e) => {
         var mydata = e.detail;
         data = mydata;
         data = data;
         posts = data.posts.filter(x=>x.category==mydata.category)
      });

  });
  
  async function hydrate(category){
    
    // make sure navigo updates the links on the page after the page has rendered
    await tick();
    
    window.category = category;
    router.updatePageLinks()
  }
  
  function getPage(page){
   console.log("curpage: "+page)
  }
  
  function slugify(str, id){
    
        if(str.length < 3){
          str = str+"-"+id;
        }
  
        str = str.replace(/^\s+|\s+$/g, ''); // trim
        str = str.toLowerCase();
      
        // remove accents, swap ñ for n, etc
        var from = "àáäâèéëêìíïîòóöôùúüûñç·/_,:;";
        var to   = "aaaaeeeeiiiioooouuuunc------";
        for (var i=0, l=from.length ; i<l ; i++) {
            str = str.replace(new RegExp(from.charAt(i), 'g'), to.charAt(i));
        }
    
        str = str.replace(/[^a-z0-9 -]/g, '') // remove invalid chars
            .replace(/\s+/g, '-') // collapse whitespace and replace by -
            .replace(/-+/g, '-'); // collapse dashes
    
        return str;
        
        
          
        
  }

</script>


 <div class="container">
<header class="text-center">
  <h1><a href="/" data-navigo>Frontsome.</a></h1>

  <div class="intro">
    <p>Awesome tools for front-end developers</p>
  </div>

  <nav>
    {#if data}
      {#each data.categories as item}
      {#if item.slug=='home'}
      <a href="/" data-navigo>{item.title}</a>
      {:else}
      <a href="/{item.slug}" data-navigo>{item.title}</a>
      {/if}
      {/each}
    {/if}
  </nav>
</header>

<div class="main mt-5" id="page">

  
 {#if posts}
   {#each posts as item}


     <section class="editable" id="{item.id}">
       
       {#if item.image}<img src="{item.image}" class="mb-4 w-100" />{/if}
       
     <h2><a href="/article/{slugify(item.title, item.id)}" data-navigo>{item.title}</a></h2>
     {@html item.body}
     </section>
   {/each}
 {/if}
 
 {#if notfound}
  <h2>Error 404</h2>
  <p>Not Found</p>
 {/if}
 
</div>
</div>
