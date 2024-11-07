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

    <!-- 학년 선택 -->
    <div class="selectBox">
      <select id="grade" v-model="selectedGrade" @change="selectGrade">
        <option value="1학년">1학년</option>
        <option value="2학년">2학년</option>
        <option value="3학년">3학년</option>
      </select>
    </div>

    <!-- 추가/삭제/저장 -->
    <div class="buttons">
      <button @click="addSubject">추가</button>
      <button @click="deleteSubject">삭제</button>
      <button @click="saveSubject">저장</button>
    </div>

    <!-- 오류 메시지 출력 -->
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

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
        <!-- 과목 목록 -->
        <!-- 정렬 -->
        <tr v-for="(subject, index) in subjects.sort((a, b) => {
          const isuComparison = a.isu.localeCompare(b.isu);
          if (isuComparison !== 0) return isuComparison;

          const pilsuComparison = a.pilsu.localeCompare(b.pilsu);
          if (pilsuComparison !== 0) return pilsuComparison;

          return a.name.localeCompare(b.name);
        })" :key="index" :class="{ selected: selectedIndex === index }" @click="selectSubject(index, $event)">
          <td>{{ subject.isu }}</td>
          <td>{{ subject.pilsu }}</td>
          <td class="subjectName">{{ subject.name }}</td>
          <td>{{ subject.credit }}</td>
          <!-- 학점이 1일 때 공백으로 나오도록 조건 분기 -->
          <td v-if="subject.credit != 1">{{ subject.attendance }}</td>
          <td v-if="subject.credit != 1">{{ subject.assignment }}</td>
          <td v-if="subject.credit != 1">{{ subject.midterm }}</td>
          <td v-if="subject.credit != 1">{{ subject.final }}</td>
          <td v-if="subject.credit != 1">{{ calTotal(subject) }}</td>
          <td v-if="subject.credit != 1"></td>
          <td v-if="subject.credit === 1"></td>
          <td v-if="subject.credit === 1"></td>
          <td v-if="subject.credit === 1"></td>
          <td v-if="subject.credit === 1"></td>
          <td v-if="subject.credit === 1"></td>
          <td v-if="subject.credit === 1"></td>
          <!-- 학점이 1일 경우 P/NP -->
          <td v-if="subject.credit === 1">{{ subject.grade }}</td>
          <td v-else :class="{ 'f-grade': calGrade(subject) === 'F' }">{{ calGrade(subject) }}</td>
        </tr>

        <!-- 과목 추가 -->
        <tr v-if="addingSubject">
          <td>
            <select v-model="newSubject.isu">
              <option value="교양">교양</option>
              <option value="전공">전공</option>
            </select>
          </td>
          <td>
            <select v-model="newSubject.pilsu">
              <option value="필수">필수</option>
              <option value="선택">선택</option>
            </select>
          </td>
          <td><input v-model="newSubject.name" placeholder="과목명 입력"></td>
          <td>
            <!-- 학점이 1일 경우를 위해 체크 -->
            <input v-model.number="newSubject.credit" type="number" min="1" @change="checkCredit">
          </td>
          <!-- 학점이 1이 아닐 경우만 입력 받음 -->
          <td v-if="newSubject.credit != 1">
            <input v-model.number="newSubject.attendance" type="number" min="0" max="20">
          </td>
          <td v-if="newSubject.credit != 1">
            <input v-model.number="newSubject.assignment" type="number" min="0" max="20">
          </td>
          <td v-if="newSubject.credit != 1">
            <input v-model.number="newSubject.midterm" type="number" min="0" max="30">
          </td>
          <td v-if="newSubject.credit != 1">
            <input v-model.number="newSubject.final" type="number" min="0" max="30">
          </td>
          <td></td>
          <td></td>
          <td></td>

          <!-- 학점이 1일때만 성적 선택 -->
          <td v-if="newSubject.credit == 1"></td>
          <td v-if="newSubject.credit == 1"></td>
          <td v-if="newSubject.credit == 1"></td>
          <td v-if="newSubject.credit == 1">
            <select v-model="newSubject.grade">
              <option value="P">Pass</option>
              <option value="NP">Non Pass</option>
            </select>
          </td>
        </tr>
      </tbody>

      <!-- 합계/총첨/평균/성적 -->
      <tfoot>
        <tr>
          <td colspan="3">합계</td>
          <td>{{ totalCredit }}</td>
          <td>{{ totalAttendance }}</td>
          <td>{{ totalAssignment }}</td>
          <td>{{ totalMidterm }}</td>
          <td>{{ totalFinal }}</td>
          <td>{{ totalScore }}</td>
          <td>{{ totalAverage }}</td>
          <td :class="{ 'f-grade': totalGrade === 'F' }">{{ totalGrade }}</td>
        </tr>
      </tfoot>
    </table>
  </div>
</template>

<script>
export default {
  name: 'ReportCard',

  data() {
    return {
      // 학년 선택: 기본값 1학년으로 설정
      selectedGrade: "1학년",

      // 기존 과목 리스트
      subjects: [],
      // 새로운 과목 입력을 위한 데이터 객체
      newSubject: {
        isu: '교양',
        pilsu: '선택',
        name: '',
        credit: 1,
        attendance: 0,
        assignment: 0,
        midterm: 0,
        final: 0,
      },
      // 과목 추가 입력란 표시 여부
      addingSubject: false,
      // 오류 메시지를 저장할 변수
      errorMessage: '',
      // 선택된 과목 인덱스
      selectedIndex: null,
    };
  },


  methods: {
    // 입력된 학점 확인
    checkCredit() {
      if (this.newSubject.credit <= 0) {
        // 오류 메시지 설정
        this.errorMessage = '학점을 입력해주세요.';
      } else {
        //오류 메세지 초기화
        this.errorMessage = '';
      }
    },

    // 새로운 과목 추가
    addSubject() {
      // 새로운 과목을 subjects 배열에 추가
      if (this.addingSubject) {

        // 과목명이 입력되지 않은 경우 예외 처리
        if (!this.newSubject.name.trim()) {
          this.errorMessage = '과목명을 입력해주세요.';
          return;
        }

        // 중복 과목 검사
        const existingSubject = this.subjects.find(subject => subject.name === this.newSubject.name);
        // F일 경우 중복 허용
        if (existingSubject && existingSubject.grade !== 'F') {
          this.errorMessage = '이미 존재하는 과목명입니다.';
          return;
        }

        this.subjects.push({ ...this.newSubject });

        // 새로운 과목 입력란 초기화
        this.newSubject = {
          isu: '교양',
          pilsu: '선택',
          name: '',
          credit: 1,
          attendance: 0,
          assignment: 0,
          midterm: 0,
          final: 0,
        };
      }
      // 추가 입력란 표시
      this.addingSubject = true;
    },

    // 입력한 과목 저장
    saveSubject() {
      // 과목명이 입력되지 않은 경우 예외 처리
      if (!this.newSubject.name.trim()) {
        // 오류 메시지 설정
        this.errorMessage = '과목명을 입력해주세요.';
        return;
      } else {
        //과목명이 제대로 입력된 경우 메세지 초기화
        this.errorMessage = '';
      }

      // 학점이 0인 경우 처리
      if (this.newSubject.credit <= 0) {
        this.errorMessage = '학점을 입력해주세요.';
        return;
      }

      // 과목명이 중복될 경우('F' 제외)
      const duplicateSubject = this.subjects.find(
        (subject) => subject.name === this.newSubject.name
      );

      if (duplicateSubject) {
        const duplicateGrade = this.calGrade(duplicateSubject);
        //console.log('이미 저장된 과목의 성적:', duplicateGrade);

        if (duplicateGrade !== 'F') {
          this.errorMessage = '이미 존재하는 과목명입니다.';
          return;
        }
      }

      // subjects 배열에 추가
      if (this.addingSubject) {
        // 과목 정보가 비어있지 않다면 추가
        if (this.newSubject.name.trim()) {
          this.subjects.push({ ...this.newSubject });
          // 새로운 과목 입력란 초기화
          this.newSubject = {
            isu: '교양',
            pilsu: '선택',
            name: '',
            credit: 0,
            attendance: 0,
            assignment: 0,
            midterm: 0,
            final: 0,
          };
        }
      }
      // 추가 입력란 숨김
      this.addingSubject = false;
    },

    //선택한 과목 삭제
    deleteSubject() {
      // 선택된 인덱스가 null이 아닐 경우 삭제
      if (this.selectedIndex !== null) {
        this.subjects.splice(this.selectedIndex, 1);
         // 선택 초기화
        this.selectedIndex = null; 
      } else {
        // 선택한 과목이 없을 경우 에러 메세지 출력
        this.errorMessage = '삭제할 과목을 선택해주세요.';
      }
    },

     // 과목 선택
     selectSubject(index) {
      //console.log('선택된 목록의 인덱스: ', index);
      console.log(event.target.className);
      this.selectedIndex = index;
    },

    // 총점 계산
    calTotal(sub) {
      // 저장된 과목이 없을 경우
      if (!sub) return 0;
      const total = Number(sub.attendance) + Number(sub.assignment) + Number(sub.midterm) + Number(sub.final);
      // 총점을 0 이상 100 이하로 제한
      return Math.max(0, Math.min(total, 100));
    },

    // 평균 계산
    calAverage() {
      // 학점이 1이 아닌 과목들만 필터링
      const validSubjects = this.subjects.filter(subject => subject.credit !== 1);
      const totalScore = validSubjects.reduce((sum, subject) => sum + this.calTotal(subject), 0);
      return validSubjects.length > 0 ? totalScore / validSubjects.length : 0;
    },

    // 성적 계산
    calGrade(subject) {
      const total = this.calTotal(subject);
      if (total >= 95) return 'A+';
      if (total >= 90) return 'A0';
      if (total >= 85) return 'B+';
      if (total >= 80) return 'B0';
      if (total >= 75) return 'C+';
      if (total >= 70) return 'C0';
      if (total >= 65) return 'D+';
      if (total >= 60) return 'D0';
      return 'F';
    }
  },

  watch: {
    // 학점이 0일때 예외처리
    'newSubject.credit'(value) {
      if (value < 1) {
        this.newSubject.credit = 1;
        this.errorMessage = '학점은 1 이상이어야 합니다.';
      } else {
        this.errorMessage = '';
      }
    },

    // 점수 입력 범위 제한
    'newSubject.attendance'(value) {
      if (value < 0) this.newSubject.attendance = 0;
      else if (value > 20) this.newSubject.attendance = 20;
    },
    'newSubject.assignment'(value) {
      if (value < 0) this.newSubject.assignment = 0;
      else if (value > 20) this.newSubject.assignment = 20;
    },
    'newSubject.midterm'(value) {
      if (value < 0) this.newSubject.midterm = 0;
      else if (value > 30) this.newSubject.midterm = 30;
    },
    'newSubject.final'(value) {
      if (value < 0) this.newSubject.final = 0;
      else if (value > 30) this.newSubject.final = 30;
    }
  },

  computed: {
    // 각 점수의 총합 계산
    totalCredit() {
      return this.subjects.reduce((sum, subject) => {
        // 학점이 1일 경우 총합에서 제외
        return subject.credit !== 1 ? sum + subject.credit : sum;
      }, 0);
    },
    totalAttendance() {
      return this.subjects.reduce((sum, subject) => sum + subject.attendance, 0);
    },
    totalAssignment() {
      return this.subjects.reduce((sum, subject) => sum + subject.assignment, 0);
    },
    totalMidterm() {
      return this.subjects.reduce((sum, subject) => sum + subject.midterm, 0);
    },
    totalFinal() {
      return this.subjects.reduce((sum, subject) => sum + subject.final, 0);
    },
    totalScore() {
      return this.totalAttendance + this.totalAssignment + this.totalMidterm + this.totalFinal;
    },
    // 평균: 전 과목의 총점의 합/전 과목 수 
    totalAverage() {
      const average = this.calAverage();
      return average;
    },
    //최종 성적은 최종 평균으로 계산
    totalGrade() {
      const average = this.calAverage()
      if (average >= 95) return 'A+';
      if (average >= 90) return 'A0';
      if (average >= 85) return 'B+';
      if (average >= 80) return 'B0';
      if (average >= 75) return 'C+';
      if (average >= 70) return 'C0';
      if (average >= 65) return 'D+';
      if (average >= 60) return 'D0';
      return 'F';
    },
  }

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

.buttons {
  float: right;
  width: 20%;
}

button {
  margin: 3px 0 3px 3px;
  width: 30%;
  height: 30%;
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
  background-color: lightsteelblue
}

/* 과목명만 왼쪽 정렬 */
.subjectName {
  text-align: left;
}

/* 홀수 줄 배경색 지정 */
tbody tr:nth-child(odd) {
  background-color: aliceblue;
}

/* 짝수 줄 배경색 지정 */
tbody tr:nth-child(even) {
  background-color: hwb(208 80% 2%);
}

/* 성적이 F일 경우 */
.f-grade {
  color: red;
}

tfoot {
  background-color: lightsteelblue
}

/* 에러 메세지 */
.error {
  border: 2px solid red;
  width: 50%;
  color: red;
  margin: 10px auto;
  text-align: center;
  padding: 5px;
  font-size: 16px;
}

/* 선택된 행에 파란색 테두리 적용 */
.selected {
  border: 1px solid blue !important;
}
</style>
