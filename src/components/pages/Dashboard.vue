<script setup>
    import Grid from '../Grid.vue';
    import { gymHealthFacts } from '../../utils';

    const props = defineProps({
        handleSelectedWorkout: Function,
        firstIncompleteWorkoutIndex: Number
    })
    
    const randomNumber = Math.floor(Math.random() * gymHealthFacts.length);
    const todaysFacts = gymHealthFacts[randomNumber];
</script>

<template>
  <section id="dashboard">
    <div class="card tip-container">
        <h2>Welcome Soldier</h2>
        <div>
            <p class="tip"><strong>Daily Tip</strong><br/>{{ todaysFacts }}</p>
        </div>
        <button @click="() => handleSelectedWorkout(firstIncompleteWorkoutIndex < 0 ? 0 : firstIncompleteWorkoutIndex)" class="">Start Workout &rarr;</button>
    </div>
    <Grid v-bind="props" />

  </section>
</template>

<style scoped>
    .tip-container,
    .tip-container div,
    #dashboard {
        display: flex;
    }

    .tip-container,
    #dashboard {
        flex-direction: column;
    }

    #dashboard {
        gap: 2rem;
    }

    .tip-container {
        gap: 0.5rem;
    }

    @media (min-width: 640px) {
        .tip-container {
            gap: 1rem;
        }

        .tip-container {
            gap: 1rem;
        }
    }
</style>