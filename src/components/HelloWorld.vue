<script setup>
  import { Authenticator, useAuthenticator } from '@aws-amplify/ui-vue';
  import '@aws-amplify/ui-vue/styles.css';
  import { defineProps, ref } from 'vue';
//  import { fetch } from 'node-fetch'
  import { API } from 'aws-amplify'

  const auth = useAuthenticator();
  const debug_mode = false;

  defineProps({msg: String});

  const ID = ref('0');
  const Situation = ref('リスタートをクリックして下さい。');
  const Choice1 = ref('なし');
  const Choice2 = ref('なし'); 
  const Result = ref('なし');
  const CorrectChoice = ref('0');
  const SelectedChoice = ref('0');
  const GetResult = ref('なし');
  const GetStatus = ref('0');
  const GetURL = ref('0');
  const GetError = ref('0');

  // 選択肢1、選択肢2、リスタートをクリックした時の条件分岐 
  const selectChoice = (playerChoice) => { 
    SelectedChoice.value = playerChoice;
    //リスタートを選択したらゲームスタート時の項目をDynamoDBから取得 
    if (playerChoice === '0') { 
        ID.value = 1; 
        Situation.value = "状態変化1";
        callAPI(); 
    //正解の選択肢なら次の段階の項目をDynamoDBから取得 
    } else if ((CorrectChoice.value != '0') && (playerChoice == CorrectChoice.value)) { 
        ID.value = ID.value + 1; 
        callAPI(); 
    //不正解なら結果を表示 
    } else if ((CorrectChoice.value != '0') && (playerChoice != CorrectChoice.value)) { 
        ID.value = 0; 
        Situation.value = Result.value; 
        Choice1.value = ''; 
        Choice2.value = ''; 
        Result.value = ''; 
        CorrectChoice.value = '0'; 
        ID.value = CorrectChoice;
    } 
  };

  //APIの呼び出しを定義する 
  const callAPI = () => { 
    //idは現在の次の段階を代入 
    var id = ID.value; 
    let url = new URL("https://osztepu9jh.execute-api.ap-northeast-1.amazonaws.com/dev/items/" + id);
    GetURL.value = url;
    let param = {
      ID: id,
    };
    url.search = new URLSearchParams(param).toString();
    // API経由でのLambdaからのレスポンスをVueインスタンスに代入する 
    const APIname = "escapegameAPI";
    const APIPath = "/items/" + id;
    const myInit = { // OPTIONAL
        headers: {}, // OPTIONAL
        response: true, // OPTIONAL (return the entire Axios response object instead of only response.data)
        queryStringParameters: {  // OPTIONAL
            name: 'param',
        },
    };
    var response_data = '';

    API.get(APIname, APIPath, myInit)
    .then(response => {
      GetStatus.value = response.status;
      GetResult.value = response.data;
      response_data = JSON.stringify(response.data);
      ID.value = JSON.parse(response_data)['ID']; 
      Situation.value = JSON.parse(response_data)['Situation']; 
      Choice1.value = JSON.parse(response_data)['Choice1']; 
      Choice2.value = JSON.parse(response_data)['Choice2']; 
      Result.value = JSON.parse(response_data)['Result']; 
      CorrectChoice.value = JSON.parse(response_data)['Correctchoice']; 
      console.log('error', ID.value);
    })
    .catch(error => {
      GetError.value = error;
      console.log('error', error);
    })
  };


</script>

<template>
  <authenticator />
  <template v-if="auth.route === 'authenticated'">
    <h1>Hello {{ auth.user?.username }}!</h1>
    <button @click="auth.signOut">Sign out</button>
  </template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
    <p>部屋から脱出せよ！</p> 
    <div id="Situation1"> 
        <p v-if="debug_mode">{{ID}}:{{CorrectChoice}}</p> 
        <p>{{Situation}}</p> 
        <p v-if="debug_mode">{{Result}}</p> 
        <p>選択肢１:{{Choice1}}</p> 
        <button type="button" @click="selectChoice('1')">選択肢１を選ぶ</button> 
        <p>選択肢２:{{Choice2}}</p> 
        <button type="button" @click="selectChoice('2')">選択肢２を選ぶ</button> 
        <p><button type="button" @click="selectChoice('0')">リスタート</button></p> 
        <p v-if="debug_mode">前回の選択肢: {{SelectedChoice}}</p>
        <p v-if="debug_mode">GETのURL: {{GetURL}}</p>
        <p v-if="debug_mode">GETステータス: {{GetStatus}}</p>
        <p v-if="debug_mode">GETエラー: {{GetError}}</p>
        <p v-if="debug_mode">GETの結果: {{GetResult}}</p>
    </div>
</template>

<script>

//  selectChoice('0');

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
