<script>
  import { createEventDispatcher } from "svelte";
  import Leaderboard from "./Leaderboard.svelte";
  import tier from "../functions/tier.js";

  export let show;
  export let schoolCode;
  export let schoolName;
  
  const updateTime = 1000 * 10;
  
  let token = "";
  let schoolCount = -1;
  let showSchoolCount = -1;
  let totalCount = -1;
  let showTotalCount = -1;
  let schoolRank = -1;
  let count = localStorage.getItem("count") || 0;
  let sendCount = 0;
  let l, p = 0;
  let secretMode = localStorage.getItem("secretMode") === "true" || false;
  
  
  const changeSecretMode = () => {
    secretMode = !secretMode;
    localStorage.setItem("secretMode", secretMode);
  }
  
  const random = () => Math.floor(Math.random() * 5);

  const pop = () => {
    if (!show) return ;
    
    count++;
    sendCount++;
    localStorage.setItem("count", count);
    p = (p + 1) % 2;
  }

  document.addEventListener("keydown", (e) => {
    if (!e.repeat) pop();
  });

  document.addEventListener("pointerdown", () => {
    pop();
  });
  
  setInterval(() => {
    if (schoolCode && sendCount && show && !l.showValue()) {
      sendPop();
    }
  }, updateTime);

  let newS = -1;
  $: {
    (show) ?
      setTimeout(() => { firstPop(); }, 0) :
      newS = totalCount = showTotalCount = schoolCount = showSchoolCount = schoolRank = -1;
  }

  const updatePop = (num) => {
    num = parseInt(num);
    if (schoolCount == -1) {
      schoolCount = num;
      showSchoolCount = num;
      return ;
    }
    schoolCount = num;
    
    const deltaCount = schoolCount - showSchoolCount;
    const time = updateTime / deltaCount; 
    const c = setInterval(() => {
      if (newS === -1) {
        clearInterval(c);
        return ;
      }

      if (showSchoolCount >= schoolCount) {
        clearInterval(c);
        return ;
      }
      showSchoolCount++;
    }, time);
  }

  const updateTotal = (num) => {
    num = parseInt(num);
    if (totalCount == -1) {      
      totalCount = num;
      showTotalCount = num;
      return ;
    }
    totalCount = num;

    const deltaCount = totalCount - showTotalCount;
    const time = updateTime / deltaCount; 
    const c = setInterval(() => {
      if (newS === -1) {
        clearInterval(c);
        return ;
      }

      if (showTotalCount >= totalCount) {
        clearInterval(c);
        return ;
      }
      showTotalCount++;
    }, time);
  }

  const isNumber = (num) => {
    if (num === null) return false;
    num = parseInt(num);
    if (num < 0) return false;
    return true;
  }

  const firstPop = async () => {
    const url = `https://port-0-kschool-backend-37y7e24l7jiwra5.gksl1.cloudtype.app/first/?schoolCode=${schoolCode}`;
    const response = await fetch(url, { method: "GET" });

    if (response.status === 200) {
      const data = await response.json();
      newS = 1;
 
      if (isNumber(data.rank)) schoolRank = data.rank;
      if (isNumber(data.pop)) updatePop(data.pop);
      if (isNumber(data.total)) updateTotal(data.total);
    }
  }

  let toManyReq = false;
  const maxCount = 200;
  const sendPop = async () => {

    if (toManyReq) {
      toManyReq = false;
      return ;
    }

    const sentCount = (sendCount > maxCount) ? maxCount : sendCount;

    const url = `https://port-0-kschool-backend-37y7e24l7jiwra5.gksl1.cloudtype.app/pop/?count=${sentCount}&token=${token}&schoolCode=${schoolCode}`;
    const response = await fetch(url, { method: "POST" });
    
    const data = await response.json();
    
    if (response.status === 201 || response.status === 200) {
      newS = 1;
      
      if (token) sendCount = 0;
      token = data.token;
 
      if (isNumber(data.rank)) schoolRank = data.rank;
      if (isNumber(data.pop)) updatePop(data.pop);
      if (isNumber(data.total)) updateTotal(data.total);
      
      // console.log(data);
      return ;
    }

    toManyReq = true;
    token = "";
  }

  const showRank = (rank) => {
    if (rank == 1) return "🥇";
    if (rank == 2) return "🥈";
    if (rank == 3) return "🥉";
    return rank + " 위";
  }

  const comma = (x) => x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");

  const trans = () => {
    return {
      duration: 80,
      css: () => `transform: scale(1.2) rotate(${random() * 5 - 10}deg)`
    };
  }

  const dispatch = createEventDispatcher();
  const resetSchool = () => {
    dispatch("resetSchool");
  }

  const openJo = () => {
    window.open("https://www.youtube.com/@jocoding").focus();
  }
  
  const openTwitter = () => {
    window.open("https://twitter.com/_awesome_dream").focus();
  }
</script>

{#if show}
  <img src="../Sgif1@.png" alt="" style="display:none;">
  <img src="../Sgif2@.png" alt="" style="display:none;">
  <div class="pop">
    <div class="mainBtn btn-size" on:click={openTwitter}>
      🌐
    </div>
    <div class="searchSchoolBtn btn-size" on:click={resetSchool}>
      🔍
    </div>
    <div class="title">
      <h1>K-SCHOOL</h1> <img class="jo" src="../img.png" draggable="false" on:click={openJo} alt="">
    </div>
    <div class="school">
      <div class="schoolData">
        <div class="schoolRank fontSize">
          {#if schoolRank > 0}
            {#if secretMode}
              -
            {:else}
              {showRank(schoolRank)}
            {/if}
          {:else}
              ⏳ 
          {/if}
        </div>
        <div class="schoolName fontSize">
          {#if secretMode}
            학교
          {:else}
            {schoolName}
          {/if}
        </div>
        <span class="schoolTier tierIcon {tier(schoolCount, schoolRank)}">
        </span>
        <div class="schoolCount fontSize">
          {#if showSchoolCount >= 0}
            {#if secretMode}
              -
            {:else}
              {comma(showSchoolCount)}
            {/if}
          {:else}
            ⏳
          {/if}
        </div>
        <!-- <div class="secretMode fontSize" on:click={changeSecretMode}>
          👁️
        </div> -->
      </div>
    </div>
    {#key count}
      <div class="count" in:trans>
        {comma(count)}
      </div>
    {/key}
    <div class="popImage popImage-{p}"></div>
    <div class="leaderboard" on:click={() => l.changeShow()}>
      <div class="leaderboard-info">
        <div class="totalCount fontSize">
          {#if showTotalCount >= 0}
            🌎 {comma(showTotalCount)}
          {:else}
          🌎 ⏳
          {/if}
        </div>
        <div class="showLeaderboard fontSize">
          &ensp;🏆
        </div>
      </div>
      <!-- <div class="alert">학교에 간식 차를 보내준다는 소문은 거짓입니다.</div> -->
      <!-- <div class="ad">
        <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-5824103033592509"
        crossorigin="anonymous"></script>
        <ins class="adsbygoogle"
            style="display:inline-block;width:240px;height:90px"
            data-ad-client="ca-pub-5824103033592509"
            data-ad-slot="4039344712"></ins>
        <script>
            (adsbygoogle = window.adsbygoogle || []).push({});
        </script>
      </div> -->
    </div>
    <Leaderboard bind:this={l} schoolCode={schoolCode} schoolShow={secretMode}/>
  </div>
{/if}

<style>
  .leaderboard-info {
    display: flex;
    align-items: center;
    flex-direction: row;
    margin: 0 auto;
  }
  
  .ad {
    height: 50px;
    margin: 0 auto;
  }

  .popImage-0 {
    background-image: url(../Sgif1@.png);
  }

  .popImage-1 {
    background-image: url(../Sgif2@.png);
  }

  .popImage {
    position: absolute;
    margin: 0 auto;
    left: 0;
    right: 0;
    height: 50%;
    width: 100%;
    max-width: 800px;
    max-height: 800px;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    touch-action: manipulation;
    text-align: center;
    box-sizing: inherit;
    display: block;
  }

  .mainBtn {
    margin: 15px;
    user-select: none;
    cursor: pointer;
    float: left;
    font-size: 30px;
  }

  .searchSchoolBtn {
    margin: 15px;
    user-select: none;
    cursor: pointer;
    float: right;
    font-size: 30px;
  }

  .title {
    display: flex;
    justify-content : center;
  }

  .schoolData {
    margin-top: 20px;
    user-select: none;
    display: flex;
    justify-content: center;
    flex-direction: row;
  }

  .schoolRank {
    cursor: default;
    display: flex;
    align-items: center;
    text-align: center;
  }

  .schoolTier {
    cursor: default;
    display: flex;
    align-items: center;
    text-align: center;
  }

  .schoolName {
    cursor: default;
    padding: 0 8px;
    display: flex;
    align-items: center;
    text-align: center;
  }

  .schoolCount {
    cursor: default;
    padding: 0 8px;
    display: flex;
    align-items: center;
    text-align: center;
  }

  .secretMode {
    cursor: pointer;
    display: flex;
    align-items: center;
    padding: 0;
    margin: 0;
  }

  .count {
    text-align: center;
    box-sizing: inherit;
    cursor: default;
    display: block;
    width: 80%;
    margin: 20px auto;
    justify-content: center;
    user-select: none;

    text-align: center;
    color: #fff;
    -webkit-text-stroke-width: 2.5px;
    -webkit-text-stroke-color: #000;
    font-weight: 1800;
    font-size: 80px;
    word-wrap: break-word;
  }

  .leaderboard {
    cursor: pointer;
    display: flex;
    justify-content: center;
    flex-direction: column;
    position: absolute;
    bottom: 0;
    width: 80%;
    left: 50%;
    transform: translate(-50%, 0);
    margin-bottom: 30px;
  }

  .showLeaderboard {
    display: flex;
    align-items: center;
    margin: 0;
    padding: 0;
  }

  .totalCount {
    display: flex;
    align-items: center;
  }
/* 
  .alert {
    margin: 0 auto;
  } */

  .fontSize {
    user-select: none;
    font-size: 30px;
  }

  .tierIcon {
    width: 40px;
    height: 42px;
  }

  .jo {
    cursor: pointer;
    user-select: none;
    width: 100px;
    height: 100px;
  }

  .btn-size {
    margin: 15px;
    font-size: 40px;
  }

  @media (max-height: 1000px) {
    .popImage {
      max-width: 300px;
      max-height: 300px;
    }

    .fontSize {
      font-size: 18px;
    }
    
    .count {
      font-size: 60px;
      -webkit-text-stroke-width: 2px;
    }

    .btn-size {
      margin: 10px;
      font-size: 18px;
    }

    .title {
      font-size: 10px;
    }

    .tierIcon {
      width: 30px;
      height: 31.5px;
    }

    .jo {
      width: 45px;
      height: 45px;
    }
  }

  @media (max-width: 500px) {
    .fontSize {
      font-size: 18px;
    }
    
    .count {
      font-size: 60px;
      -webkit-text-stroke-width: 2px;
    }

    .btn-size {
      margin: 10px;
      font-size: 18px;
    }

    .title {
      font-size: 10px;
    }

    .tierIcon {
      width: 30px;
      height: 31.5px;
    }

    .jo {
      width: 45px;
      height: 45px;
    }
  }

  @media (max-height: 600px) {
    .popImage {
      max-width: 200px;
      max-height: 200px;
    }

    .title {
      font-size: 8px;
    }

    .count {
      font-size: 36px;
      -webkit-text-stroke-width: 1.4px;
    }

    .fontSize {
      font-size: 9px;
    }

    .btn-size {
      margin: 5px;
      font-size: 10px;
    }

    .tierIcon {
      width: 20px;
      height: 21px;
    }

    .jo {
      width: 35px;
      height: 35px;
    }
  }

  @media (max-width: 281px) {
    .title {
      font-size: 8px;
    }

    .count {
      font-size: 36px;
      -webkit-text-stroke-width: 1.4px;
    }

    .fontSize {
      font-size: 9px;
    }

    .btn-size {
      margin: 5px;
      font-size: 10px;
    }

    .tierIcon {
      width: 20px;
      height: 21px;
    }

    .jo {
      width: 35px;
      height: 35px;
    }
  }

  * {
    user-select: none;
  }

  .tierIcon {
    background-repeat: no-repeat;
    background-position: center;
    background-size: contain;
    display: inline-block;
  }

  .bronze {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%23A86350'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='%23663228'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='%2337221F' stroke-width='1.8'/%3E%3C/svg%3E");
  }

  .silver {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M15.2717 14.5676C14.4563 15.9394 13.2691 17.021 11.9131 17.6276L9.97272 11.3003L10.7472 4.48078C12.1404 4.45651 13.7761 5.31448 14.7409 6.27787C15.7058 7.24126 16.2863 8.59474 16.3831 10.1068C16.4799 11.6189 16.087 13.1957 15.2717 14.5676Z' fill='%2397928E'/%3E%3Cpath d='M2.12311 10.5356C2.23324 8.91676 2.90237 7.38641 4.01606 6.20632L9.60696 11.0138L12.567 17.5321C11.09 18.2039 8.86356 18.1922 7.29616 17.7725C5.72877 17.3527 4.35588 16.4015 3.41228 15.0814C2.46867 13.7614 2.01298 12.1545 2.12311 10.5356Z' fill='%23616161'/%3E%3Cmask id='path-3-inside-1_1_185' fill='white'%3E%3Cpath d='M3.84315 5.34315C2.5344 6.65189 1.71994 8.37394 1.53852 10.2159C1.35711 12.0578 1.81997 13.9056 2.84824 15.4446C3.87652 16.9835 5.40658 18.1183 7.17772 18.6555C8.94887 19.1928 10.8515 19.0993 12.5615 18.391C14.2714 17.6828 15.6829 16.4035 16.5554 14.7712C17.4279 13.1389 17.7074 11.2546 17.3463 9.43928C16.9852 7.624 16.0059 5.99008 14.5751 4.81592C13.1444 3.64176 11.3508 3 9.5 3L9.5 9.7471C9.78987 9.7471 10.0708 9.84761 10.2948 10.0315C10.5189 10.2154 10.6723 10.4713 10.7288 10.7556C10.7854 11.0399 10.7416 11.335 10.605 11.5906C10.4683 11.8463 10.2473 12.0466 9.97947 12.1575C9.71166 12.2685 9.41369 12.2831 9.1363 12.199C8.85892 12.1148 8.61929 11.9371 8.45825 11.6961C8.29721 11.4551 8.22472 11.1657 8.25313 10.8772C8.28154 10.5887 8.4091 10.319 8.61406 10.1141L3.84315 5.34315Z'/%3E%3C/mask%3E%3Cpath d='M3.84315 5.34315C2.5344 6.65189 1.71994 8.37394 1.53852 10.2159C1.35711 12.0578 1.81997 13.9056 2.84824 15.4446C3.87652 16.9835 5.40658 18.1183 7.17772 18.6555C8.94887 19.1928 10.8515 19.0993 12.5615 18.391C14.2714 17.6828 15.6829 16.4035 16.5554 14.7712C17.4279 13.1389 17.7074 11.2546 17.3463 9.43928C16.9852 7.624 16.0059 5.99008 14.5751 4.81592C13.1444 3.64176 11.3508 3 9.5 3L9.5 9.7471C9.78987 9.7471 10.0708 9.84761 10.2948 10.0315C10.5189 10.2154 10.6723 10.4713 10.7288 10.7556C10.7854 11.0399 10.7416 11.335 10.605 11.5906C10.4683 11.8463 10.2473 12.0466 9.97947 12.1575C9.71166 12.2685 9.41369 12.2831 9.1363 12.199C8.85892 12.1148 8.61929 11.9371 8.45825 11.6961C8.29721 11.4551 8.22472 11.1657 8.25313 10.8772C8.28154 10.5887 8.4091 10.319 8.61406 10.1141L3.84315 5.34315Z' stroke='%2355504B' stroke-width='3.6' mask='url(%23path-3-inside-1_1_185)'/%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%2397928E'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='%23616161'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='%23363434' stroke-width='1.8'/%3E%3C/svg%3E");
  }

  .gold {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M15.2717 14.5676C14.4563 15.9394 13.2691 17.021 11.9131 17.6276L9.97272 11.3003L10.7472 4.48078C12.1404 4.45651 13.7761 5.31448 14.7409 6.27787C15.7058 7.24126 16.2863 8.59474 16.3831 10.1068C16.4799 11.6189 16.087 13.1957 15.2717 14.5676Z' fill='%23E39921'/%3E%3Cpath d='M2.12311 10.5356C2.23324 8.91676 2.90237 7.38641 4.01606 6.20632L9.60695 11.0138L12.567 17.5321C11.09 18.2039 8.86356 18.1922 7.29616 17.7725C5.72876 17.3527 4.35588 16.4015 3.41227 15.0814C2.46867 13.7614 2.01297 12.1545 2.12311 10.5356Z' fill='%23C57624'/%3E%3Cmask id='path-3-inside-1_2_233' fill='white'%3E%3Cpath d='M3.84315 5.34315C2.5344 6.65189 1.71994 8.37394 1.53852 10.2159C1.35711 12.0578 1.81997 13.9056 2.84824 15.4446C3.87652 16.9835 5.40658 18.1183 7.17772 18.6555C8.94887 19.1928 10.8515 19.0993 12.5615 18.391C14.2714 17.6828 15.6829 16.4035 16.5554 14.7712C17.4279 13.1389 17.7074 11.2546 17.3463 9.43928C16.9852 7.624 16.0059 5.99008 14.5751 4.81592C13.1444 3.64176 11.3508 3 9.5 3L9.5 9.7471C9.78987 9.7471 10.0708 9.84761 10.2948 10.0315C10.5189 10.2154 10.6723 10.4713 10.7288 10.7556C10.7854 11.0399 10.7416 11.335 10.605 11.5906C10.4683 11.8463 10.2473 12.0466 9.97947 12.1575C9.71166 12.2685 9.41369 12.2831 9.1363 12.199C8.85892 12.1148 8.61929 11.9371 8.45825 11.6961C8.29721 11.4551 8.22472 11.1657 8.25313 10.8772C8.28154 10.5887 8.4091 10.319 8.61406 10.1141L3.84315 5.34315Z'/%3E%3C/mask%3E%3Cpath d='M3.84315 5.34315C2.5344 6.65189 1.71994 8.37394 1.53852 10.2159C1.35711 12.0578 1.81997 13.9056 2.84824 15.4446C3.87652 16.9835 5.40658 18.1183 7.17772 18.6555C8.94887 19.1928 10.8515 19.0993 12.5615 18.391C14.2714 17.6828 15.6829 16.4035 16.5554 14.7712C17.4279 13.1389 17.7074 11.2546 17.3463 9.43928C16.9852 7.624 16.0059 5.99008 14.5751 4.81592C13.1444 3.64176 11.3508 3 9.5 3L9.5 9.7471C9.78987 9.7471 10.0708 9.84761 10.2948 10.0315C10.5189 10.2154 10.6723 10.4713 10.7288 10.7556C10.7854 11.0399 10.7416 11.335 10.605 11.5906C10.4683 11.8463 10.2473 12.0466 9.97947 12.1575C9.71166 12.2685 9.41369 12.2831 9.1363 12.199C8.85892 12.1148 8.61929 11.9371 8.45825 11.6961C8.29721 11.4551 8.22472 11.1657 8.25313 10.8772C8.28154 10.5887 8.4091 10.319 8.61406 10.1141L3.84315 5.34315Z' stroke='%238B4700' stroke-width='3.6' mask='url(%23path-3-inside-1_2_233)'/%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%23E39921'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='%23C57624'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='%23573007' stroke-width='1.8'/%3E%3C/svg%3E");
  }

  .platinum {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M13 8L17.5 10L16 10.5V14.5L12.5 18L10 14L13 8Z' fill='%23007F59'/%3E%3Cpath d='M12.5 17.5L10.05 12L5.5 10L3 16.5L6 15C6.66269 18.7967 7.99691 18.9334 12.5 17.5Z' fill='%2300432F'/%3E%3Cmask id='path-3-inside-1_16_10' fill='white'%3E%3Cpath d='M17 12C17 13.3543 16.6071 14.6795 15.8691 15.815C15.131 16.9505 14.0794 17.8475 12.8417 18.3973C11.604 18.947 10.2334 19.126 8.89608 18.9124C7.55874 18.6988 6.31206 18.1019 5.30718 17.194L10 12H17Z'/%3E%3C/mask%3E%3Cpath d='M17 12C17 13.3543 16.6071 14.6795 15.8691 15.815C15.131 16.9505 14.0794 17.8475 12.8417 18.3973C11.604 18.947 10.2334 19.126 8.89608 18.9124C7.55874 18.6988 6.31206 18.1019 5.30718 17.194L10 12H17Z' stroke='%23002119' stroke-width='3.6' mask='url(%23path-3-inside-1_16_10)'/%3E%3Cpath d='M3 17L7 5L18 10L10 12L3 17Z' stroke='%23002119' stroke-width='1.2'/%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%23018059'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='%2300432F'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='%23002119' stroke-width='1.8'/%3E%3C/svg%3E%0A");
  }

  .diamond {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M13 8L17.5 10L16 10.5V14.5L12.5 18L10 14L13 8Z' fill='%2347EBFD'/%3E%3Cpath d='M12.5 17.5L10.05 12L5.5 10L3 16.5L6 15C6.66269 18.7967 7.99691 18.9334 12.5 17.5Z' fill='%2300C9D4'/%3E%3Cmask id='path-3-inside-1_16_33' fill='white'%3E%3Cpath d='M17 12C17 13.3543 16.6071 14.6795 15.8691 15.815C15.131 16.9505 14.0794 17.8475 12.8417 18.3973C11.604 18.947 10.2334 19.126 8.89608 18.9124C7.55874 18.6988 6.31206 18.1019 5.30718 17.194L10 12H17Z'/%3E%3C/mask%3E%3Cpath d='M17 12C17 13.3543 16.6071 14.6795 15.8691 15.815C15.131 16.9505 14.0794 17.8475 12.8417 18.3973C11.604 18.947 10.2334 19.126 8.89608 18.9124C7.55874 18.6988 6.31206 18.1019 5.30718 17.194L10 12H17Z' stroke='%23008088' stroke-width='3.6' mask='url(%23path-3-inside-1_16_33)'/%3E%3Cpath d='M3 17L7 5L18 10L10 12L3 17Z' stroke='%2329AAA1' stroke-width='1.2'/%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%2347EBFD'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='%2300C9D4'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='%23008088' stroke-width='1.8'/%3E%3C/svg%3E%0A");
  }

  .none {
    background-image: url("data:image/svg+xml,%3Csvg width='20' height='21' viewBox='0 0 20 21' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M10.5 13L6 3L17 13H10.5Z' fill='%23DCDCDC'/%3E%3Cpath d='M10.5 13L6 3V18L10.5 13Z' fill='white'/%3E%3Cpath d='M6 18V3L17 13H10.5L6 18Z' stroke='black' stroke-width='1.8'/%3E%3C/svg%3E%0A");
  }
</style>