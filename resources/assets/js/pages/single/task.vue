<template>
    <div class="electron-task">
        <PageTitle :title="taskInfo.name"/>
        <Loading v-if="loadIng > 0"/>
        <TaskDetail v-else :task-id="taskInfo.id" :open-task="taskInfo"/>
    </div>
</template>

<style lang="scss" scoped>
.electron-task {
    height: 100%;
    display: flex;
    flex-direction: column;
    overflow: auto;
    .task-detail {
        flex: 1;
        margin: 0;
        padding: 18px 22px;
        border-radius: 0;
    }
}
</style>
<style lang="scss">
.electron-task {
    .task-detail {
        .task-info {
            .head {
                .function {
                    margin-right: 0;
                    .open {
                        display: none;
                    }
                }
            }
        }
    }
}
</style>
<script>
import TaskDetail from "../manage/components/TaskDetail";
import {mapState} from "vuex";
export default {
    components: {TaskDetail},
    data() {
        return {
            loadIng: 0,
            taskId: 0,
        }
    },
    mounted() {
        //
    },
    computed: {
        ...mapState(['cacheTasks']),

        taskInfo() {
            return this.cacheTasks.find(({id}) => id === this.taskId) || {}
        }
    },
    watch: {
        '$route': {
            handler() {
                this.getInfo();
            },
            immediate: true
        },
    },
    methods: {
        getInfo() {
            this.taskId = $A.runNum(this.$route.params.id);
            if (this.taskId <= 0) {
                return;
            }
            this.loadIng++;
            this.$store.dispatch("getTaskOne", {
                task_id: this.taskId,
                archived: 'all'
            }).then(() => {
                this.loadIng--;
                this.$store.dispatch("getTaskContent", this.taskId);
                this.$store.dispatch("getTaskFiles", this.taskId);
                this.$store.dispatch("getTaskForParent", this.taskId).catch(() => {})
            }).catch(({msg}) => {
                this.loadIng--;
                $A.modalError({
                    content: msg,
                    onOk: () => {
                        if (this.$Electron) {
                            window.close();
                        }
                    }
                });
            });
        }
    }
}
</script>
