<script setup>
  import Welcome from './components/pages/Welcome.vue';
  import LogoutButton from './components/LogoutButton.vue';
  import Layout from './components/layouts/Layout.vue';
  import Dashboard from './components/pages/Dashboard.vue';
  import Workout from './components/pages/Workout.vue';

  import { ref, computed, onMounted } from 'vue';
  import { workoutProgram } from './utils';
  import { Clerk } from '@clerk/clerk-js';

  const clerk = new Clerk(import.meta.env.VITE_CLERK_PUBLISHABLE_KEY);

  // Auth-Status zentral verwalten
  const isSignedIn = ref(false);
  const isAuthReady = ref(false);

  clerk.load().then(() => {
    isSignedIn.value = !!clerk.user;
    clerk.addListener(({ user }) => {
      isSignedIn.value = !!user;
    });
    if (!clerk.user) {
      setTimeout(() => {
        const el = document.getElementById('sign-in');
        if (el) clerk.mountSignIn(el);
        isAuthReady.value = true;
      }, 0);
    } else {
      isAuthReady.value = true;
    }
  });

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
    return isCompleteCheck;
  }); 

  const firstIncompleteWorkoutIndex = computed(() => {
    const allWorkouts = data.value
    if(!allWorkouts) {
        return -1;
      }
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

  function handleResetPlan() {
    selectedDisplay.value = 2
    selectedWorkout.value = -1; // Deselect a workout
    data.value = defaultData; // Reset the data to the initial state
    localStorage.removeItem('workouts'); // Clear the local storage
  }
  
  onMounted(() => {
    if (!localStorage) {return}
    if (localStorage.getItem('workouts')) {
      const saveData = JSON.parse(localStorage.getItem('workouts'));
      data.value = saveData || defaultData; // Load saved workouts or use default data
      selectedDisplay.value = 2; // Default to Dashboard page
    }
  });

  function goToDashboard() {
    selectedDisplay.value = 2; // Dashboard anzeigen
    selectedWorkout.value = -1; // Workout deselektieren
  }
</script>

<template>
  <header>
    <LogoutButton :clerk="clerk" v-if="isSignedIn" />
  </header>

  <Layout>
    <!-- Clerk Login Widget, wenn nicht eingeloggt -->
    <div v-if="!isSignedIn">
      <div id="sign-in"></div>
    </div>
    <!-- App-Seiten nur wenn eingeloggt -->
    <template v-else>
      <Welcome
        :handleChangeDisplay="handleChangeDisplay"
        v-if="selectedDisplay == 1"
      />
      <Dashboard
        :handleResetPlan="handleResetPlan"
        :firstIncompleteWorkoutIndex="firstIncompleteWorkoutIndex"
        :handleSelectedWorkout="handleSelectedWorkout"
        v-if="selectedDisplay == 2"
      />
      <Workout
        @save="goToDashboard"
        :isWorkoutComplete="isWorkoutComplete"
        :data="data"
        :selectedWorkout="selectedWorkout"
        v-if="workoutProgram?.[selectedWorkout]"
      />
    </template>
  </Layout>
</template>