<script setup>
    import { ref, computed } from 'vue';
    const emit = defineEmits(['save']);
    import Portal from '../Portal.vue';
    import { exerciseDescriptions, workoutProgram } from '../../utils';

    const workoutType = [
        'Push',
        'Pull',
        'Legs'
    ];

    const { data, selectedWorkout} = defineProps({
        data: Object,
        selectedWorkout: Number,
        handleSaveWorkout: Function,
        isWorkoutComplete: Boolean
    });
    
    const { workout, warmup } = workoutProgram[selectedWorkout];
    // let selectedExercise = null;
        let selectedExercise = ref(null);
            const exercises = ref([]);

            // Dynamisch über die API laden (deutsche Namen und Beschreibungen)
            import { onMounted } from 'vue';
            onMounted(async () => {
                // Lade Namen
                const aliasRes = await fetch('https://wger.de/api/v2/exercisealias/?language=1&limit=100');
                const aliasData = await aliasRes.json();
                // Lade Beschreibungen
                const infoRes = await fetch('https://wger.de/api/v2/exerciseinfo/?language=1&limit=100');
                const infoData = await infoRes.json();
                // Verbinde die Daten über den Namen und hole die Beschreibung aus translations
                exercises.value = aliasData.results.map(alias => {
                    // Finde das passende Info-Objekt, das eine Übersetzung mit gleichem Namen und Sprache 1 (Deutsch) hat
                    const info = infoData.results.find(i =>
                        i.translations.some(t => t.name === alias.alias && t.language === 1)
                    );
                    // Hole die Übersetzung
                    const translation = info?.translations.find(t => t.language === 1 && t.name === alias.alias);
                    return {
                        id: alias.id,
                        name: alias.alias,
                        description: translation?.description || ''
                    };
                });
            });
        const weightInputError = ref({});
    const exerciseDescription = computed (() => exerciseDescriptions[selectedExercise.value  ] || 'No description available for this exercise.');

    function validateWeightInput(event, key) {
        const value = event.target.value;
        if (!/^\d*$/.test(value)) {
            weightInputError.value[key] = "Please enter digits only.";
            event.target.value = value.replace(/\D/g, "");
        } else {
            weightInputError.value[key] = "";
        }
    }
    
    function handleCloseModal() {
        selectedExercise.value = null
    }
    
        function handleSaveWorkout() {
                // Beispiel: UserId und WorkoutType ggf. anpassen!
                const payload = {
                    userId: 'demoUser',
                    date: new Date(),
                    workoutType: workoutType[selectedWorkout % 3],
                    exercises: workout.map(w => ({
                        name: w.name,
                        sets: w.sets,
                        reps: w.reps,
                        weight: data[selectedWorkout][w.name] || 0
                    }))
                };
                fetch('http://localhost:3002/workout', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                })
                .then(res => res.json())
                .then(result => {
                    // Optional: Erfolgsmeldung, z.B. console.log(result)
                    emit('save');
                })
                .catch(err => {
                    // Optional: Fehlerbehandlung
                    console.error(err);
                });
            }
</script>

<template>
  <Portal :handleCloseModal="handleCloseModal"  v-if="selectedExercise">
        <div class="exercise-description">
            <h3>{{ selectedExercise }}</h3>
            <div>
                <small>Description</small>
                <p>{{ exerciseDescription }}</p>
            </div>
            <button @click="handleCloseModal">Close <i class="fa-solid fa-xmark"></i></button>
        </div>
  </Portal>  
  <!-- <pre>{{ exercises }}</pre> -->
  <section id="workout-card">
    <div class="plan-card card">
      <div class="plan-card-header">
        <p>Day {{ selectedWorkout < 9 ? '0' + (selectedWorkout + 1) : (selectedWorkout + 1) }}</p>
        <!-- <button class="reset-button">Reset Workout</button> -->
        <i class="fa-solid fa-dumbbell"></i>
      </div>
      <h2>{{ workoutType[selectedWorkout % 3] }} Workout</h2>
      <!-- <p>{{ workout.description }}</p> -->
    </div>
    <div class="workout-grid">
        <h4 class="grid-name">Warmup</h4>
        <h6 class="">Sets</h6>
        <h6 class="">Reps</h6>
        <h6 class="grid-weights">Weights</h6>
        <div class="workout-grid-row" v-for="(w, wIdx) in warmup" :key="wIdx">
            <div class="grid-name">
                <select v-model="w.name" class="exercise-dropdown">
                    <option v-for="ex in exercises" :key="ex.id" :value="ex.name">{{ ex.name }}</option>
                </select>
                <button @click="selectedExercise.value = w.name">
                    <i class="fa-regular fa-circle-question"></i>
                </button>
            </div>
                <select v-model="w.sets" class="sets-dropdown">
                    <option v-for="n in 100" :key="'sets-' + n" :value="n">{{ n }}</option>
                </select>
                <select v-model="w.reps" class="reps-dropdown">
                    <option v-for="n in 100" :key="'reps-' + n" :value="n">{{ n }}</option>
                </select>
            <input class="grid-weights" placeholder="14kg" type="text" @input="event => validateWeightInput(event, 'warmup-' + wIdx)" />
            <div v-if="weightInputError['warmup-' + wIdx]" class="input-error">{{ weightInputError['warmup-' + wIdx] }}</div>
        </div>
        <div class="workout-grid-line"></div>
        <h4 class="grid-name">Workout</h4>
        <h6 class="">Sets</h6>
        <h6 class="">Reps</h6>
        <h6 class="grid-weights">Weights</h6>
        <div class="workout-grid-row" v-for="(w, wIdx) in workout" :key="wIdx">
            <div class="grid-name">
                                <select v-model="w.name" class="exercise-dropdown">
                                    <option v-for="ex in exercises" :key="ex.id" :value="ex.name">{{ ex.name }}</option>
                                </select>
                    <button @click="() => {
                        selectedExercise = w.name
                    }">
                        <i class="fa-regular fa-circle-question"></i>
                    </button>
            </div>
                <select v-model="w.sets" class="sets-dropdown">
                    <option v-for="n in 100" :key="'sets-' + n" :value="n">{{ n }}</option>
                </select>
                <select v-model="w.reps" class="reps-dropdown">
                    <option v-for="n in 100" :key="'reps-' + n" :value="n">{{ n }}</option>
                </select>
            <input v-model="data[selectedWorkout][w.name]" class="grid-weights" placeholder="14kg" type="text" @input="event => validateWeightInput(event, 'workout-' + wIdx)" />
            <div v-if="weightInputError['workout-' + wIdx]" class="input-error">{{ weightInputError['workout-' + wIdx] }}</div>
        </div>
    </div>
        <!-- <div v-if="weightInputError" class="input-error">{{ weightInputError }}</div> -->
    <div class="card workout-btns">
        <button @click="handleSaveWorkout">Save & Exit <i class="fa-solid fa-save"></i></button>
        <button :disabled="!isWorkoutComplete" @click="handleSaveWorkout">Complete <i class="fa-solid fa-check"></i></button>
    </div>
  </section>
</template>

<style scoped>
    #workout-card,
    .plan-card {
        display: flex;
        flex-direction: column;
    }

    #workout-card {
        gap: 1.5rem;
    }

    .plan-card-header,
    .workout-btns {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 1rem;
    }

    .workout-grid,
    .workout-grid-row {
        display: grid;
        grid-template-columns: repeat(7, minmax(0, 1fr));
        gap: 1rem;
    }

    .workout-grid-row,
    .workout-grid-line {
        grid-column: span 7 / span 7;
    }

    .workout-grid-line {
        margin: 0.5rem 0;
        height: 3px;
        border-radius: 2px;
        background: var(--background-muted);
    }

    .grid-name {
        grid-column: span 3 / span 3;
        display: flex;
        align-items: center;
        gap: 1rem;
    }

    .grid-name button {
        padding: 0;
        border: none;
        box-shadow: none;
    }

    .grid-name button:hover {
        transform: none;
        box-shadow: none;
        color: var(--color-link);
    }

    .workout-grid-row .grid-name button {
        opacity: 0;
        pointer-events: none;
    }

    .workout-grid-row:hover .grid-name button {
        opacity: 1;
        pointer-events: all;
    }

    .grid-name p {
        text-transform: capitalize;
    }

    .grid-weights {
        grid-column: span 2 / span 2;
    }

    .workout-btns button {
        flex: 1;
    }

    .workout-btns button i {
        padding-left: 0.5rem;
    }

    .exercise-description {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .exercise-description h3 {
        text-transform: capitalize;
    }

    .exercise-description button i {
        padding-left: 0.5rem;
    }

    .input-error {
        font-size: 0.95rem;
        color: red;
        margin-top: 0.5rem;
        grid-column: span 7;
    }
</style>