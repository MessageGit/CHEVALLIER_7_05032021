<template>
    <div class="post-content" v-if="!isDeleted">
        <div class="post-profil-img">
            <img :src="dataPost.ownerAvatar" :class="{'no-avatar' : dataPost.ownerAvatar == './default-avatar.png' }" alt="Image de profil">
        </div>
        <div class="post-infos">
            <span class="post-owner">Posté le <b>{{ postDate }}</b> à <b>{{ postTime }}</b><br />par <b>{{ dataPost.ownerName }}</b></span>
            <div class="post-manage comments-nb" v-if="!inEdit">
                <img src="@/assets/icons/comments-icon.png" alt="Commentaires liés à ce post">
                <span>0</span>
            </div>
            <div class="post-manage" @click="editPost()" v-if="userData.isAdmin || dataPost.ownerId == userData.id">
                <img src="@/assets/icons/edit-icon.png" alt="Editer ce post">
                <span v-if="inEdit">Valider</span>
            </div>
            <div class="post-manage" @click="deletePost()" v-if="userData.isAdmin && !inEdit || dataPost.ownerId == userData.id && !inEdit">
                <img src="@/assets/icons/delete-icon.png" alt="Supprimer ce post">
            </div>
        </div>
        <div class="parag-post">
            <textarea class="textarea-edit" spellcheck="false" v-model="editText" v-if="inEdit"></textarea>
            {{ postText }}
            <img :src="dataPost.fileImg" class="image-post" v-if="dataPost.fileImg" alt="Image jointe au post">
        </div>
    </div>
</template>

<script>
import store from '@/modules/store.json'

export default {
    name: 'postForum',
    props: ['userData', 'userToken', 'dataPost'],
    data() {
        return {
            postDate: this.newDateFormat(),
            postTime: this.newTimeFormat(1), // Fuseaux horaire (UTC+1)
            postText: this.dataPost.txt,
            editText: this.dataPost.txt, inEdit: false,
            isDeleted: false,
        }
    },
    methods: {
        newDateFormat() {
            let newDate = this.dataPost.createdAt.split('T')[0];
            newDate = newDate.split('-')[2] + '/' + newDate.split('-')[1] + '/' + newDate.split('-')[0];
            return newDate;
        },
        newTimeFormat(timeZone) {
            let newTime = parseInt(this.dataPost.createdAt.substring(11, 13))+timeZone;
            newTime = newTime + this.dataPost.createdAt.substring(13, 16).replace(':', 'h');
            return newTime;
        },
        editPost() {
            if(this.inEdit) { 
                this.inEdit = false;
                const data = { txt: this.editText }
                fetch(store.host_api + '/post/' + this.dataPost.id, { 
                        method: 'PUT',
                        headers: { 'Content-type' : 'application/json', 'Authorization' : 'Bearer ' + this.userToken },
                        body: JSON.stringify(data)
                    })
                    .then(response => {
                        if(response.status == 201) { return response.json()
                        } else { throw 'Le post n\'a pas pu être modifié.' }
                    })
                    .then(data => { console.log(data.message); this.postText = this.editText; })
                    .catch((err) => { console.log(err) });
            } else { this.inEdit = true }
        },
        deletePost() {
            fetch(store.host_api + '/post/' + this.dataPost.id, { method: 'DELETE', headers: { 'Content-type' : 'application/json', 'Authorization' : 'Bearer ' + this.userToken } })
                .then(response => {
                    if(response.status == 201) { return response.json()
                    } else { throw 'Le post n\'a pas pu être supprimé.' }
                })
                .then(data => { console.log(data.message); this.isDeleted = true; })
                .catch((err) => { console.log(err) });
        }
    }
}
</script>

<style scoped>
.post-content {
    position: relative;
    margin-top: 65px;
    width: 100%; min-height: 90px;
    border: 2px solid transparent;
    border-top: 12px solid #e2e2e2;
}

.post-content .post-profil-img {
    position: absolute;
    top: -35px; left: 45px;
    width: 90px; height: 90px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: white;
    border-left: 30px solid white;
    border-right: 30px solid white;
    z-index: 1;
}

.post-content .post-profil-img img {
    position: relative;
    width: 100%; height: 100%;
    border: 2px solid #cecece;
    border-radius: 10px;
}

.post-content .post-profil-img .no-avatar { 
    border: none;
    bottom: 15px;
}

.post-content .post-infos {
    position: absolute;
    right: 0px;
    width: 72%;
    height: 55px;
    text-align: left;
    align-items: center;
}

.post-content .post-infos .post-owner {
    position: relative;
    font-size: 14px;
    float: left;
    top: 10px;
}

.post-content .post-infos .post-manage {
    position: relative;
    float: right;
    top: 10px;
    height: 38px;
    display: flex;
    align-items: center;
    cursor: pointer;
}

.post-content .post-infos .comments-nb {
    width: 70px;
    padding-left: 0px;
    top: 12px;
}

.post-content .post-infos .comments-nb span {
    margin-top: 0px!important;
}

.post-content .post-infos .comments-nb img {
    margin-left: 15px!important;
}

.post-content .post-infos .post-manage:hover {
    background-color: #f7f7f7;
    border-radius: 45px;
}

.post-content .post-infos .post-manage img {
    margin-left: 10px;
    margin-right: 10px;
    width: 20px;
}

.post-content .post-infos .post-manage span {
    margin-top: 3px;
    font-size: 13px;
    font-weight: bold;
    margin-left: 5px;
    margin-right: 10px;
    margin-bottom: 2px;
    color: #9577d4;
}

.post-content .parag-post {
    position: relative;
    margin-top: 90px;
    left: 5%; width: 90%;
    padding-bottom: 25px;
    text-align: left;
    white-space: pre-line;
}

.post-content .parag-post .textarea-edit {
    position: absolute;
    top: 0px; left: 0px;
    width: 100%; height: 100%;
    outline: 0; border: none;
    font-family: Nunito;
    font-weight: bold;
}

.post-content .parag-post .image-post {
    position: relative;
    margin-top: 30px;
    width: 100%;
    border-radius: 18px;
    opacity: 0.5;
    transition-duration: 0.5s;
}

.post-content .parag-post .image-post:hover {
    opacity: 1;
}
</style>