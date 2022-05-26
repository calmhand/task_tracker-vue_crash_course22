<template>
    <AddTask 
        v-show="showAddTask" 
        @add-task="addTask"
    />

    <UserTasks 
        @toggle-reminder="toggleReminder" 
        @delete-task="deleteTask" 
        :tasks="tasks"
    />
</template>

<script>
    import UserTasks from '../components/Tasks'
    import AddTask from '../components/AddTask'
    export default {
        name: 'Home',
        props: {
            showAddTask: Boolean,
        },
        components: {
            UserTasks,
            AddTask,
        },
        data() {
            return {
                tasks: [],
            }
        },
        methods: {
            async addTask(task) {
                // send task object to server
                const res = await fetch('api/tasks', {
                    method: 'POST',
                    headers: {
                        'Content-type': 'application/json',
                    },
                    body: JSON.stringify(task)
                })

                const data = await res.json()

                this.tasks = [...this.tasks, data]
            },

            async deleteTask(id) {
                if (confirm('Are you sure?')) {
                    const res = await fetch(`/api/tasks/${id}`, {
                        method: 'DELETE'
                    })

                    res.status === 200 ? (this.tasks = 
                    this.tasks.filter((task) => task.id !== id))
                    : alert('Deletion Error')
                }
            },

            async toggleReminder(id) {
                const tasktoToggle = await this.fetchTask(id)
                const updTask = {...tasktoToggle, reminder: !tasktoToggle.reminder}

                const res = await fetch(`api/tasks/${id}`, {
                    method: 'PUT',
                    headers: {
                        'Content-type': 'application/json'
                    },
                    body: JSON.stringify(updTask)
                })

                const data = await res.json()

                this.tasks = this.tasks.map((task) => 
                task.id === id ? 
                { ...task, reminder: 
                data.reminder } : task
                )
            },

            async fetchTasks() {
                const res = await fetch('api/tasks') // get request

                const data = await res.json()

                return data
            },

            async fetchTask(id) {
                const res = await fetch(`api/tasks/${id}`) // get request for single task

                const data = await res.json()

                return data
            }
        },
        async created() {
            this.tasks = await this.fetchTasks()
        }
    }
</script>