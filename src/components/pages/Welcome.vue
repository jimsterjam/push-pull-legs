<script setup>
    defineProps({
        handleChangeDisplay: Function
    });
</script>

<template>
  <section id="welcome">
    <div class="benefits">
        <h2>Workout for everyday<br/>if you want ...</h2>
        <div class="">
            <p>✅ This way you can do 6 sessions per week </p>
            <p>✅ Get strong and absolutely shredded. </p>
        </div>
        <div>
            <h3>The Training Plan</h3>
                <p>This training plan follows a structurte known as 
                <strong>Bro Split</strong></p>
                <h5><i>Push &rarr; Pull &rarr; Legs &rarr; Repeat</i></h5>
        </div>
        <div class="card challenge">
            <h3>🔥 Start here 🔥</h3>
            <p>Ready to go workouts!</p>
            <p>Easy to adjust!</p>
            <button @click=" () => handleChangeDisplay(2) ">Let's go&rarr;</button>
        </div>
    </div>
  </section>
</template>

<style scoped>
    #welcome,
    .challenge,
    .benefits {
        display: flex;
        flex-direction: column;
    }   
    
    #welcome {
        gap: 1.5rem;
    }

    .benefits {
        gap: 0.5rem;
    }

    .challenge {
        gap: 0.25rem;
    }

    @media (min-width: 640px) {
        #welcome {
            gap: 2rem;
            padding: 2rem 0;
        }

        .benefits {
            gap: 1rem;
        }
        
    }


</style>