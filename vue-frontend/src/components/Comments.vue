<template>
    <form @submit.prevent="createComment" v-show="!editComment">
        <input name="comment" placeholder="Publiez un commentaire" v-model="newComment" class="text">                           
        <input type="submit" value="Je commente!" class="btn">
    </form>
    <p>{{errMsg}}</p>
    <div id="commentsContainer" v-show="comments.length > 0 ">
        <div :key="comment.id" v-for="comment in comments" class="comment">
            <div class="header">
                <div class="profileContainer">
                        <img :src="'http://localhost:3000/images/' + comment.User.profile" :alt="comment.User.profile" class="profile">>    
                </div>
                <div class="commentUsername">
                    <h5>{{comment.User.firstname}} {{comment.User.lastname}}</h5>     
                </div>
                <div v-if="auth(comment.User.id)" class="optionsBtn">
                    <button v-if="editComment != comment.id" @click="toggleComment(comment.id)"><i class="far fa-edit modify"></i>modifier</button>
                    <button v-if="editComment == comment.id" @click="toggleComment(comment.id)"><i class="fas fa-arrow-left"></i>annuler</button>
                    <button @click="deleteComment(comment.id)"><i class="far fa-trash-alt delete"></i>supprimer</button>    
                </div>               
            </div>                       
            <p v-show="!editComment" class="commentText">{{comment.text}}</p>
            <form v-if="editComment == comment.id" @submit.prevent="modifyComment(comment.id)">
                <input name="updateComment" ref="modify" :value="comment.text" class="text">                           
                <input type="submit" value="Je modifie!" class="btn">
            </form>                          
        </div>        
    </div>
</template>

<script>
export default {
    name: 'Comments', 
    props: {
        comments: Array,
        isAdmin: Boolean,
        userId: Number,
        postId: Number
    },
    data() {
        return {
            newComment: null,
            updateComment: null,
            editComment: false,
            errMsg: null
        }
    },
    methods: {
        createComment() {
            if (!this.newComment) {
                this.errMsg = "Erreur => vous devez remplir le champ <commentaire> pour créer un nouveau commentaire!"
                return
            }
            const data = {
                text: this.newComment,
                userId: JSON.stringify(this.userId),
                postId: JSON.stringify(this.postId)
            }
            fetch(`http://localhost:3000/api/comments/create`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    Authorization: `Bearer ${localStorage.getItem('token')}`
                },
                mode: 'cors',
                body: JSON.stringify(data)
            })  
                .then(res => res.json())
                .then(data => this.$emit('created', data.id))
                .catch(error => {console.error(error)})
            this.newComment = ''
        },
        auth(commentUserId) {
            if (this.isAdmin) {
                return true
            }
            if (this.userId !== commentUserId) {
                return false
            }
            return true              
        },
        toggleComment(commentId, cancel) {
            if (cancel) {
                console.log("hello")
            }
            if (this.editComment == commentId) {
                commentId = null
            }
            this.editComment = commentId
        },
        deleteComment(commentId) {
            if (confirm("êtes vous sûr de vouloir supprimer ce commentaire ?")) {
                fetch(`http://localhost:3000/api/comments/${JSON.stringify(commentId)}`, {
                    method: 'DELETE',
                    headers: {
                        Authorization: `Bearer ${localStorage.getItem('token')}`
                    } 
                })
                    .catch(error => console.log(error))
                this.$emit('deleted', commentId)
            }   
        },
        modifyComment(commentId) {        
            const data = {
                text: this.$refs.modify.value
            }
            fetch(`http://localhost:3000/api/comments/${JSON.stringify(commentId)}`, {
                method: 'PUT',
                headers: {
                    Authorization: `Bearer ${localStorage.getItem('token')}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data) 
            })
                .catch(error => console.log(error))
            this.$emit('modified', commentId, data.text)
            this.toggleComment(commentId)
        }
    },
    emits: ['created', 'deleted', 'modified']
}
</script>

<style scoped>
#commentsContainer {
    margin: auto;
    border-radius: 4px;
    overflow-wrap: break-word;
}
.comment {
    border-bottom: 1px solid rgb(0 0 0 / 10%);
    margin: 2rem;
    border-radius: 4px;
}
.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
.commentUsername {
    display: flex;
    justify-content: space-between;
    vertical-align: center;
    width: 90%;
    padding-left: 16px;
}
.commentText {
    margin: 1rem 0 1rem 0;
}
.profileContainer {
    margin: auto;
    max-width: 48px;
    max-height: 48px;
    height: 48px;
    width: 48px;
    min-height: 48px;
    min-width: 48px;
    border-radius: 50%;
    overflow: hidden;
}
.profile {
    height: 100%;
    width: 100%;
    object-fit: cover;
}
form {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
}
input {
    margin-top: 1.8rem;
    width: 100%;
    padding: 10px;
    box-shadow: 2px 2px 8px 5px rgb(0 0 0 / 10%);
    border-style: none;
    border-radius: 4px;
    outline: none;
}
.optionsBtn {
    display: flex;
}
button {
    background: none;
    border-style: none;
    outline: none;
    margin-left: 1rem;
    display: flex; 
}
.btn {
    width: 30%;
}
.btn:hover {
    box-shadow: 2px 2px 8px 5px #00000033;
}
.btn:active {
    transform: scale(0.98);
}
i {
    margin-right: 3px;
}
@media screen and (max-width: 992px) {
    .header {
        flex-wrap: wrap;
        justify-content: space-around;
        line-height: 2rem;
    }
    .profileContainer {
        margin: 0;
    }
    .commentUsername {
        display: flex;
        justify-content: center;
        width: 100%;
        padding-left: 0;    
    }
    .optionsBtn {
        width: 100%;
        justify-content: space-between;
    }
    .commentText {
        background-color: rgba(0, 0, 0, 0.05);
        border-radius: 4px;
        padding: 0.5rem;
    }
    .btn {
        width: 100%;
    }
}
</style>