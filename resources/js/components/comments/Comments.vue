<template>
    <div>
        <template v-if="reply">
            <comment-reply :comment="reply" />
        </template>
        <template v-else>
            <h3 class="mb-5">{{ meta.total }} comments</h3>
            <new-comment
                :endpoint="endpoint"
            />
            <template v-if="comments.length">
                <ul class="list-unstyled">
                    <comment v-for="comment in comments" :key="comment.id" :comment="comment" />
                </ul>
            </template>
            <p v-else class="mt-4">No comments to display</p>

            <a
                href="#"
                class="btn btn-light btn-block"
                @click.prevent="loadMore"
                v-if="meta.current_page < meta.last_page"
            >Show More</a>
        </template>
    </div>
</template>

<script>
    import Comment from './Comment'
    import NewComment from './NewComment'
    import CommentReply from './CommentReply'
    import bus from '../../bus'
    import axios from 'axios'

    export default {
        data () {
            return {
                comments: [],
                meta: {},
                reply: null
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
            Comment,
            CommentReply
        },
        methods: {
            async loadComments (page = 1) {
                let comments = await axios.get(`${this.endpoint}?page=${page}`)

                this.comments = comments.data.data
                this.meta = comments.data.meta
            },
            async fetchMeta () {
                let comments = await axios.get(`${this.endpoint}?page=${this.meta.current_page}`)

                this.meta = comments.data.meta
            },
            async loadMore () {
                let comments = await axios.get(`${this.endpoint}?page=${this.meta.current_page + 1}`)

                this.comments.push(...comments.data.data)
                this.meta = comments.data.meta
            },
            async prependComment(comment) {
                this.comments.unshift(comment)

                await this.fetchMeta()

                if (this.meta.current_page < this.meta.last_page) {
                    this.comments.pop()
                }
            },
            setReplying (comment) {
                this.reply = comment
            },
            appendReply({ comment, reply }) {
                console.log(this.comment.children);
                _.find(this.comments, { id: comment.id }).children.push(reply)
            }
        },
        mounted () {
            this.loadComments()

            bus.$on('comment:stored', this.prependComment)
            bus.$on('comment:reply', this.setReplying)
            bus.$on('comment:reply-cancelled', () => this.reply = null)

            bus.$on('comment:replied', this.appendReply)
        }
    }
</script>
