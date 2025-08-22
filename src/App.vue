<script setup>
  import Welcome from './components/pages/Welcome.vue';
  import Layout from './components/layouts/Layout.vue';
  import Dashboard from './components/pages/Dashboard.vue';
  import Workout from './components/pages/Workout.vue';

  import { ref, computed, onMounted } from 'vue';
  import { workoutProgram } from './utils';

  
  const defaultData = {}
    for (let workoutIdx in workoutProgram) {
      const workoutData = workoutProgram[workoutIdx];
      defaultData[workoutIdx] = {}; // Initialize the workout object for each workout

      for (let e of workoutData.workout) {
        defaultData[workoutIdx][e.name] = ''; // Initialize each exercise with an empty string
      }
    }
  
  const selectedDisplay = ref(1);
  const data = ref(defaultData)
  const selectedWorkout = ref(-1); 

  const isWorkoutComplete = computed(() => {
    const currWorkout = data.value[selectedWorkout.value];
    if (!currWorkout) return false;

    const isCompleteCheck = Object.values(currWorkout).every(ex => !!ex); // Check if all exercises have a value
    console.log('ISCOMPLETE: ', isCompleteCheck);
    return isCompleteCheck;
  }); 

  const firstIncompleteWorkoutIndex = computed(() => {
    const allWorkouts = data.value
    if(!allWorkouts) {
        return -1;
      }
    
    // loop over key value pair and check if completed

    for (const [index, workout] of Object.entries(allWorkouts)) {
      const isComplete = Object.values(workout).every(ex => !!ex);
      if (!isComplete) {
        return parseInt(index); // Return the first incomplete workout index
      }
    }
    return -1; // Return -1 if all workouts are complete
  })
  

  function handleChangeDisplay(idx) {
    selectedDisplay.value = idx
  }

  function handleSelectedWorkout(idx) {
    selectedDisplay.value = 3;
    selectedWorkout.value = idx;
  }

  function handleSaveWorkout() {
    localStorage.setItem('workouts', JSON.stringify(data.value));

    selectedDisplay.value = 2; // Default to Welcome page

    selectedWorkout.value = -1; // Deselect a workout
  }

  function handleResetPlan() {
    selectedDisplay.value = 2
    selectedWorkout.value = -1; // Deselect a workout
    data.value = defaultData; // Reset the data to the initial state
    localStorage.removeItem('workouts'); // Clear the local storage
  }
  
  onMounted(() => {
    //console.log('App mounted');
    if (!localStorage) {return}
    if (localStorage.getItem('workouts')) {
      const saveData = JSON.parse(localStorage.getItem('workouts'));
      data.value = saveData || defaultData; // Load saved workouts or use default data
      selectedDisplay.value = 2; // Default to Dashboard page
    }
    
  });

</script>

<template>
  <Layout>
  <!-- Page1 -->
    <Welcome :handleChangeDisplay="handleChangeDisplay" v-if="selectedDisplay == 1"/>
  <!-- Page2 -->
    <Dashboard :handleResetPlan="handleResetPlan" :firstIncompleteWorkoutIndex="firstIncompleteWorkoutIndex" :handleSelectedWorkout="handleSelectedWorkout" v-if="selectedDisplay == 2"/>
  <!-- Page3 -->
    <Workout :handleSaveWorkout="handleSaveWorkout" :isWorkoutComplete="isWorkoutComplete" :data="data" :selectedWorkout="selectedWorkout" v-if="workoutProgram?.[selectedWorkout]"/>
  </Layout>
</template>

<style scoped>

</style>
