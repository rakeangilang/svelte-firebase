<script>
    import { initializeApp } from "firebase/app";
    import {
        getFirestore,
        collection,
        onSnapshot,
        doc,
        addDoc,
        updateDoc,
        deleteDoc,
    } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";
    // import { browser } from "$app/env";

    // const firebaseApp =
    //     browser &&
    //     (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());
    // const db = browser && getFirestore();

    const firebaseApp = initializeApp(firebaseConfig);

    const db = getFirestore();

    const colRef = collection(db, "todos");

    let todos = [];

    const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
        let fbTodos = [];
        querySnapshot.forEach((doc) => {
            let todo = { ...doc.data(), id: doc.id };
            fbTodos = [todo, ...fbTodos];
        });

        fbTodos.sort((a, b) => a.createdAt - b.createdAt);

        todos = fbTodos;
    });

    console.log({ firebaseApp, db });

    let task = "";
    let error = "";

    const addTodo = async () => {
        if (task != "") {
            const docRef = await addDoc(collection(db, "todos"), {
                task: task,
                isComplete: false,
                createdAt: new Date(),
            });
            error = "";
        } else {
            error = "Task is empty";
        }
        task = "";
    };

    const markAsCompleted = async (item) => {
        await updateDoc(doc(db, "todos", item.id), {
            isComplete: !item.isComplete,
        });
    };

    const deleteTodo = async (id) => {
        await deleteDoc(doc(db, "todos", id));
    };

    const keyIsPressed = (event) => {
        if (event.key === "Enter") {
            addTodo();
        }
    };

    // $: console.table(todos);
</script>

<div class="main-container">
    <input type="text" placeholder="New task...." bind:value={task} />
    <button on:click={addTodo}>Add</button>

    <ol>
        {#each todos as item}
            <!-- class complete akan berlaku ketika isComplete true -->
            <li class:complete={item.isComplete}>
                <span>
                    {item.task}
                </span>
                <span>
                    <button on:click={() => markAsCompleted(item)}>✓</button>
                    <button on:click={() => deleteTodo(item.id)}>✖</button>
                </span>
            </li>
        {:else}
            <p>No todos yet.</p>
        {/each}
        <p class="error">{error}</p>
    </ol>
</div>

<svelte:window on:keydown={keyIsPressed} />

<style>
    .complete {
        text-decoration: line-through;
    }

    .error {
        color: red;
    }

    .main-container {
        display: block;
        width: 500px;
        margin: 0 auto;
        border: 1px solid;
        padding: 10px;
        box-shadow: 3px 6px #888888;
    }
</style>
