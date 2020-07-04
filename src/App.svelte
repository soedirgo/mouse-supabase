<script>
 import { onMount } from 'svelte';
 import { createClient } from '@supabase/supabase-js';

 const supabase = createClient(
     __supabaseUrl__,
     __supabaseKey__,
 );

 let canvas,
     ctx,
     coord = { x: 0, y: 0 },
     ptr = new Image(20, 20),
     numPlayers = 1,
     id = Math.floor(Math.random() * (2**63));

 ptr.src = 'pointer.png';

 onMount(async () => {
     await supabase.from('cursors').insert([{ id, x: 0, y: 0 }]);

     ctx = canvas.getContext('2d');
     let frame;

     (async function loop() {
         await new Promise(r => setTimeout(r, 100));

         frame = requestAnimationFrame(loop);

         await supabase.from('cursors').eq('id', id).update(coord);

         let { body: cursors } = await supabase.from('cursors').neq('id', id).select('*');
         numPlayers = cursors.length + 1
         ctx.clearRect(0, 0, canvas.width, canvas.height);
         for (let {x, y} of cursors) {
             ctx.drawImage(ptr,
                           x - 5,
                           y - 2,
                           20, 20);
         }
     })();

     return () => {
         cancelAnimationFrame(frame);
     }
 })

 async function handleMousemove(e) {
     coord = { x: e.offsetX, y: e.offsetY };
 }

 async function handleUnload() {
     await supabase.from('cursors').eq('id', id).delete();
 }
</script>

<main>
    <canvas
        bind:this={canvas}
        width={500}
        height={479}
        on:mousemove={handleMousemove}
    >
    </canvas>
    <p>No. of players: {numPlayers}</p>
    <p>Coordinates: {coord.x} x {coord.y}</p>
</main>

<svelte:window on:unload={handleUnload}/>

<style>
 main {
     text-align: center;
     padding: 1em;
     max-width: 240px;
     margin: 0 auto;
 }

 h1 {
     color: #ff3e00;
     text-transform: uppercase;
     font-size: 4em;
     font-weight: 100;
 }

 canvas {
     border: 1px solid black;
     background-image: url(/labyrinth.jpg);
 }

 @media (min-width: 500px) {
     main {
         max-width: none;
     }
 }
</style>
