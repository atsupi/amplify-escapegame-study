<template>
  <authenticator>
  </authenticator>
  <template v-if="auth.route === 'authenticated'">
    <h1>Hello {{ auth.user?.username }}!</h1>
    <button @click="auth.signOut">Sign out</button>
  </template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.
    </p>
  </div>
    <p>部屋から脱出せよ！</p> 
    <div id="Situation1"> 
        <p>{{Situation}}</p> 
        <p>選択肢１:{{Choice1}}</p> 
        <button type="button" onclick="selectChoice('1')">選択肢１を選ぶ</button> 
        <p>選択肢２:{{Choice2}}</p> 
        <button type="button" onclick="selectChoice('2')">選択肢２を選ぶ</button> 
        <p><button type="button" onclick="selectChoice('0')">リスタート</button></p> 
    </div> 
</template>

<script setup>
  import { createApp } from 'vue'
  import { Authenticator, useAuthenticator } from '@aws-amplify/ui-vue';
  import '@aws-amplify/ui-vue/styles.css';
  const auth = useAuthenticator();
</script>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
        //DynamoDBから受け取った情報を表示するためのVueインスタンスを生成します 
        const vm = {
          data() {
            return {
                ID:'0', 
                Situation:'', 
                Choice1:'', 
                Choice2:'', 
                Result:'', 
                CorrectChoice:'' 
            }
          }
        }
        createApp(vm).mount("#Situation1");
        // 選択肢1、選択肢2、リスタートをクリックした時の条件分岐 
        const selectChoice = (playerChoice)=>{ 
            //リスタートを選択したらゲームスタート時の項目をDynamoDBから取得 
            if (playerChoice == '0') { 
                vm.ID = 1; 
                callAPI(); 
            //正解の選択肢なら次の段階の項目をDynamoDBから取得 
            } else if ((vm.CorrectChoice != '0') && (playerChoice == vm.CorrectChoice)) { 
                vm.ID = vm.ID + 1; 
                callAPI(); 
            //不正解なら結果を表示 
            } else if ((vm.CorrectChoice != '0') && (playerChoice != vm.CorrectChoice)) { 
                vm.ID = 0; 
                vm.Situation = vm.Result; 
                vm.Choice1 = ''; 
                vm.Choice2 = ''; 
                vm.Result = ''; 
                vm.CorrectChoice = '0'; 
            } 
        } 
        selectChoice('1');
        //APIの呼び出しを定義する 
        var callAPI = ()=>{ 
            //idは現在の次の段階を代入 
            var id = vm.ID; 
            // ヘッダーオブジェクトの生成 
            var myHeaders = new Headers(); 
            myHeaders.append("Content-Type", "application/json"); 
            //Lambda側に渡す値の設定 
            var raw = JSON.stringify({"ID":id}); 
            // JSONオブジェクトの生成 
            var requestOptions = { 
                method: 'GET', 
                headers: myHeaders, 
                body: raw, 
                redirect: 'follow' 
            }; 
            // API経由でのLambdaからのレスポンスをVueインスタンスに代入する 
            fetch("https://osztepu9jh.execute-api.ap-northeast-1.amazonaws.com/dev/items/" + id, requestOptions) 
            .then(response => response.text()) 
            .then(result => { 
                vm.ID = JSON.parse(result).body['ID']; 
                vm.Situation = JSON.parse(result).body['Situation']; 
                vm.Choice1 = JSON.parse(result).body['Choice1']; 
                vm.Choice2 = JSON.parse(result).body['Choice2']; 
                vm.Result = JSON.parse(result).body['Result']; 
                vm.CorrectChoice = JSON.parse(result).body['Correctchoice']; 
                return 0; 
            }) 
            .catch(error => console.log('error', error)); 
        } 
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
