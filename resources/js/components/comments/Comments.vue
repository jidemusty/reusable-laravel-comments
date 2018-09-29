<template>
    <div>
        <new-comment
            :endpoint="endpoint"
        />
        <template v-if="comments.length">
            <ul class="list-unstyled">
                <comment v-for="comment in comments" :key="comment.id" :comment="comment" />
            </ul>
        </template>
        <p v-else class="mt-4">No comments to display</p>
    </div>
</template>

<script>
    import Comment from './Comment'
    import NewComment from './NewComment'
    import bus from '../../bus'
    import axios from 'axios'

    export default {
        data () {
            return {
                comments: []
            }
        },
        props: {
            endpoint: {
                required: true,
                type: String
            }
        },
        components: {
            NewComment,
            Comment
        },
        methods: {
            async loadComments (page = 1) {
                let comments = await axios.get(`${this.endpoint}?page=${page}`)
                this.comments = comments.data.data
            },
            prependComment(comment) {
                console.log(comment)
            }
        },
        mounted () {
            this.loadComments()
            bus.$on('comment:stored', this.prependComment)
        }
    }
</script>
