<template>
  <div class="intro">
    <h3></h3>
    <p>
      [] 2024 Front-End 개발실습 <strong> 과제 1. 학점관리 사이트 만들기</strong>
    </p>
  </div>

  <div class="reportCard">
    <br>
    <h1>[ 학점 관리 ]</h1>
    <br>

    <div class="selectBox">
      <select id="grade" v-model="selectedGrade" @change="selectGrade">
        <option value="1학년">1학년</option>
        <option value="2학년">2학년</option>
        <option value="3학년">3학년</option>
      </select>
    </div>

    <table>
      <thead>
        <tr>
          <th>이수</th>
          <th>필수</th>
          <th>과목명</th>
          <th>학점</th>
          <th>출석점수</th>
          <th>과제점수</th>
          <th>중간고사</th>
          <th>기말고사</th>
          <th>총점</th>
          <th>평균</th>
          <th>성적</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
          <td></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'ReportCard',
}
</script>

<style scoped>
.intro {
  border: 3px solid gray;
  width: 50%;
  margin: auto;
}

.reportCard {
  width: 90%;
  margin: auto;
}

.selectBox {
  margin: auto;
  float: left;
}

table {
  width: 100%;
  margin: auto;
  border-collapse: collapse;
  font-size: large;
}

th,
td {
  border: 1px solid black;
  padding: 5px;
  text-align: center;
}

th {
  background-color: hwb(208 75% 3%);
}
</style>
