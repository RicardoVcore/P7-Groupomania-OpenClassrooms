<template>
    <Nav redirection="/home" :logged="true" />  
    <Header :home="false" header="Membres" />
    <div id="usersContainer">
        <div id="query">
            <span><i class="fas fa-search"></i></span>
            <input type="text" placeholder="exemple: John Doe" v-model="search">
        </div>           
        <div :key="user.id" v-for="user in filterUsers" class="user">
            <div id="header">
                <div class="profileContainer">
                    <img :src="'http://localhost:3000/images/' + user.profile" :alt="user.profile" class="profile">    
                </div>
                <h4>{{user.firstname}} {{user.lastname}}</h4>   
            </div>             
            <div>                
                <p>Nombre de post(s): #{{user.Posts.length}}</p>
                <p>Nombres total de likes reçus: #{{getLikes(user.Posts)}}</p>
                <p>Dernier post le: {{getDate(user.Posts)}}</p>                    
            </div>           
        </div>            
    </div>    
</template>

<script>
import Nav from "../components/Nav.vue"
import Header from "../components/Header.vue"
export default {
    name: 'Team',
    components: {
        Nav,
        Header
    },
    data() {
        return {
            users: [],
            showPosts: null,
            search: ""
        }
    },
    methods: {
        async fetchUsers() {
            const res = await fetch('http://localhost:3000/api/users/all')
            const data = await res.json()
            data.forEach(user => {
                user.username = user.firstname + " " + user.lastname
            })
            return data
        },
        getLikes(userPosts) {
            let totalLikes = 0
            userPosts.forEach(post => {
                totalLikes += post.Likes.length
            })
            return totalLikes
        },
        getDate(userPosts) {
            if (userPosts < 1) {
                return "ce user n'a pas encore posté"
            } else {
                return userPosts[userPosts.length -1].createdAt.split('T')[0]   
            }
            
        }
    },
    async created() {
        this.users = await this.fetchUsers()
    },
    computed: {
        filterUsers: function() {
            return this.users.filter((user) => {
                return user.username.toLowerCase().match(this.search.toLowerCase())
            })
        }
    }
}
</script>

<style scoped>
#usersContainer {
    max-width: 600px;
    margin: auto;
    border-radius: 4px;
}
.user {
    box-shadow: 2px 2px 8px 5px rgb(0 0 0 / 10%);
    margin: 1rem;
    padding: 1rem;
    border-radius: 4px;
}
#header {
    display: flex;
    align-items: center;
    margin-bottom: 1rem;;
}
.profileContainer {
    width: 96px;
    height: 96px;
    min-width: 64px;
    min-height: 64px;    
    border-radius: 50%;
    overflow: hidden;
    margin-right: 1rem;
}
.profile {
    height: 100%;
    width: 100%;
    object-fit: cover;
}
p {
    margin: 0.5rem;
}
#query {
    text-align: center;
}
input {
    padding: 6px;
    border: 1px solid #1c68e6;
    border-radius: 0 4px 4px 0;
    border-left-style: none;
}
input:focus {
    outline: none;
}
span {
    padding: 6px;
    border: 1px solid #1c68e6;
    border-right-style: none;
    border-radius: 4px 0 0 4px;
}
@media screen and (max-width: 992px) {
    #usersContainer {
        max-width: 90%;
    }
}
</style>