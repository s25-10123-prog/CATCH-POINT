```react
import React, { useState, useEffect, useRef } from 'react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  createUserWithEmailAndPassword, 
  signInWithEmailAndPassword, 
  sendPasswordResetEmail, 
  onAuthStateChanged, 
  signOut,
  signInWithCustomToken,
  signInAnonymously
} from 'firebase/auth';
import { 
  getFirestore, 
  doc, 
  setDoc, 
  onSnapshot 
} from 'firebase/firestore';
import { 
  Lock, 
  LogOut, 
  Save, 
  GraduationCap, 
  BookOpen, 
  AlertCircle,
  CheckCircle2,
  Mail,
  Plus,
  Trash2,
  Sparkles,
  RefreshCw,
  Info,
  ChevronRight,
  ClipboardCheck,
  Search,
  FileText,
  UserCheck,
  Settings,
  Sun,
  Moon,
  X,
  BarChart3,
  BrainCircuit,
  Award,
  BookMarked,
  ArrowRight,
  Check,
  TrendingUp,
  TrendingDown,
  Activity
} from 'lucide-react';

// ==========================================================
// [FIREBASE CONFIGURATION]
// ==========================================================
const firebaseConfig = {
  apiKey: "AIzaSyCNncvAHxoiL4HZE4ephrST0THGPmbLfnc",
  authDomain: "app123-9a3c9.firebaseapp.com",
  projectId: "app123-9a3c9",
  storageBucket: "app123-9a3c9.firebasestorage.app",
  messagingSenderId: "933256403505",
  appId: "1:933256403505:web:a2ad048181197cc6139310",
  measurementId: "G-LJ6GL7YWVX"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'student-record-app';

// 2022 개정 교육과정 기준 고정 기입 항목
const FIXED_STRUCTURE = {
  autonomous: { 
    label: "자율활동", 
    limit: 500, 
    placeholder: "학급 회의, 행사 기획, 자치 활동 등 본인의 주도적 자율활동 내용을 입력하세요.",
    tip: "역할에 따른 주도성, 공동체 기여도, 위기 극복 능력이 드러나면 좋습니다."
  },
  club: { 
    label: "동아리활동", 
    limit: 500, 
    placeholder: "동아리 내에서의 구체적 역할, 탐구 주제, 실험 내용 등을 입력하세요.",
    tip: "수업 중 품은 의문을 탐구해 나간 심화 학습 과정을 중심으로 기록해보세요."
  },
  career: { 
    label: "진로활동", 
    limit: 700, 
    placeholder: "자신의 진로 방향성 탐색, 관련 전공 독서 및 연구 분석 활동 내용을 입력하세요.",
    tip: "단순히 '강연을 들었다'가 아닌, 강연 후 찾아본 심화 독서나 탐구 보고서가 핵심입니다."
  },
  service: { 
    label: "봉사활동 (특기사항)", 
    limit: 500, 
    placeholder: "교내외 봉사활동 중 진정성 있는 참여 태도와 배운 점을 기록하세요.",
    tip: "지속적이고 헌신적으로 참여한 활동 및 그를 통한 내적 성장을 기술하세요."
  },
  comprehensive: { 
    label: "행동특성 및 종합의견", 
    limit: 1000, 
    placeholder: "한 해를 마무리하며 교사가 작성할 추천 의견의 초안을 작성하세요.",
    tip: "담임 교사가 작성하는 부분인 만큼, 객관적이고 긍정적인 핵심 에피소드 위주로 초안을 정리합니다."
  }
};

// 사용자가 명시하여 조율한 학년별 고정 자동 탑재 교과목 데이터베이스 (기록장 기준: 공통 단일과목)
const DEFAULT_GRADE_SUBJECTS = {
  '1': [
    { name: "공통국어", limit: 500, content: "" },
    { name: "공통수학", limit: 500, content: "" },
    { name: "공통영어", limit: 500, content: "" },
    { name: "통합과학", limit: 500, content: "" },
    { name: "통합사회", limit: 500, content: "" },
    { name: "한국사", limit: 500, content: "" },
    { name: "정보", limit: 500, content: "" },
    { name: "과학탐구실험", limit: 500, content: "" },
    { name: "체육", limit: 500, content: "" },
    { name: "음악", limit: 500, content: "" },
    { name: "미술", limit: 500, content: "" },
    { name: "진로와직업", limit: 500, content: "" }
  ],
  '2': [
    { name: "문학", limit: 500, content: "" },
    { name: "독서와 작문", limit: 500, content: "" },
    { name: "대수", limit: 500, content: "" },
    { name: "미적분Ⅰ", limit: 500, content: "" },
    { name: "영어Ⅰ", limit: 500, content: "" },
    { name: "영어Ⅱ", limit: 500, content: "" },
    { name: "스포츠생활", limit: 500, content: "" }
  ],
  '3': [
    { name: "화법과 언어", limit: 500, content: "" },
    { name: "확률과 통계", limit: 500, content: "" },
    { name: "영어 독해와 작문", limit: 500, content: "" },
    { name: "스포츠 과학", limit: 500, content: "" },
    { name: "스포츠 문화", limit: 500, content: "" },
    { name: "음악 감상과 비평", limit: 500, content: "" },
    { name: "미술 창작", limit: 500, content: "" },
    { name: "생태와 환경", limit: 500, content: "" }
  ]
};

// 학년별 추가 가능한 선택과목 풀
const SELECTABLE_OPTIONAL_SUBJECTS = {
  '1': [],
  '2': [
    "인공지능 수학", "세계사", "세계시민과 지리", "사회와 문화", "현대사회와 윤리", 
    "물리학", "화학", "생명과학", "지구과학", "인공지능 기초", 
    "일본어", "중국어", "교육의 이해", "보건", "문학과 영상", 
    "기하", "영어 발표와 토론", "동아시아 역사 기행", "한국지리 탐구", 
    "법과 사회", "경제", "윤리와 사상", "역학과 에너지", 
    "물질과 에너지", "세포와 물질대사", "지구시스템과학", "일본 문화", 
    "한문", "인간과 철학"
  ],
  '3': [
    "주제 탐구 독서", "미적분2", "영미 문학 읽기", "도시의 미래탐구", 
    "정치", "인문학과 윤리", "역사로 탐구하는 현대 세계", "전자기와 양자", 
    "화학 반응의 세계", "생물의 유전", "행성우주과학", "데이터 과학", 
    "일본어", "중국어", "인간과 심리", "언어생활 탐구", 
    "실용 통계", "수학과제 탐구", "심화 영어", "미디어 영어", 
    "기후변화와 지속가능한 세계", "윤리문제 탐구", "기후변화와 환경생태", 
    "융합과학 탐구", "인간과 경제활동"
  ]
};

// 석차등급 미산출 대상 교양, 예체능, 진로선택 과목 필터링 키워드
const EXCLUDED_LIBERAL_ARTS = [
  '보건', '교육', '철학', '심리', '종교', '진로', '직업', '논리학', '논술', '환경', '실용', '연극', '영화', '미디어', '정보보호',
  '체육', '음악', '미술', '스포츠', '창작', '비평', '생태', '세계시민과 지리', '일본 문화', '한문'
];

// 대입 주요 학과 목록 데이터베이스
const UNIVERSITY_DEPARTMENTS = [
  "컴퓨터공학과", "소프트웨어학과", "인공지능학과", "전기공학과", "전자공학과", "반도체공학과", "기계공학과", "화학공학과", "신소재공학과", "생명공학과", "항공우주공학과", "건설환경공학과", "산업공학과", "융합에너지공학과",
  "수학과", "통계학과", "물리학과", "화학과", "생명과학과", "지구시스템과학과", "환경과학과", "천문우주학과", "식품영양학과",
  "의예과", "치의예과", "한의예과", "약학과", "간호학과", "수의예과", "임상병리학과", "물리치료학과",
  "국어국문학과", "영어영문학과", "중어중문학과", "불어불문학과", "독어독문학과", "사학과", "철학과", "고고인류학과", "언어학과",
  "경영학과", "경제학과", "정치외교학과", "행정학과", "사회학과", "심리학과", "미디어커뮤니케이션학과", "신문방송학과", "법학과", "사회복지학과", "지리학과", "문헌정보학과", "소비자학과",
  "교육학과", "국어교육과", "영어교육과", "수학교육과", "역사교육과", "지리교육과", "물리교육과", "화학교육과", "생물교육과", "초등교육과", "유아교육과", "특수교육과",
  "체육학과", "스포츠지도학과", "디자인학과", "회화과", "조소과", "작곡과", "성악과", "피아노과", "연극영화과"
];

export default function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [view, setView] = useState('login'); // login, signup, forgotPassword, main
  const [activeTab, setActiveTab] = useState('record'); // record (기록장), grades (성적 분석), feedback (AI 피드백)
  
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [message, setMessage] = useState({ type: '', text: '' });

  const [grade, setGrade] = useState('1'); 
  const [saveStatus, setSaveStatus] = useState(''); // '', 'saving', 'success', 'error'
  const [isAddingSubject, setIsAddingSubject] = useState(false);
  const [newSubjectName, setNewSubjectName] = useState('');
  const [byteCalcMode, setByteCalcMode] = useState('char'); // 'char' (글자수) 또는 'byte' (NEIS 바이트 기준)

  // 학과 검색 및 선택 관련 상태
  const [deptSearch, setDeptSearch] = useState('');
  const [selectedDept, setSelectedDept] = useState('컴퓨터공학과');
  const [isDeptDropdownOpen, setIsDeptDropdownOpen] = useState(false);

  // AI 피드백 결과 및 상태
  const [aiFeedbackText, setAiFeedbackText] = useState('');
  const [isGeneratingFeedback, setIsGeneratingFeedback] = useState(false);

  // 테마 및 설정 창 관리 상태
  const [isDarkMode, setIsDarkMode] = useState(() => {
    if (typeof window !== 'undefined') {
      const savedTheme = localStorage.getItem('theme');
      return savedTheme === 'dark';
    }
    return false;
  });
  const [isSettingsOpen, setIsSettingsOpen] = useState(false);

  // 모달 기반의 커스텀 알림/확인창 관리
  const [modal, setModal] = useState({ show: false, title: '', message: '', onConfirm: null });

  // 1, 2, 3학년 전체 데이터를 관리하는 통합 상태
  const [allGradesData, setAllGradesData] = useState({
    '1': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }, 
    '2': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
    '3': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }
  });

  // Canvas 레퍼런스
  const canvasRef = useRef(null);

  // 다크 모드 스타일 클래스 전파
  useEffect(() => {
    if (isDarkMode) {
      document.documentElement.classList.add('dark');
      localStorage.setItem('theme', 'dark');
    } else {
      document.documentElement.classList.remove('dark');
      localStorage.setItem('theme', 'light');
    }
  }, [isDarkMode]);

  // 이메일 유효성 검사 및 유연한 admin 처리
  const processEmailInput = (emailStr) => {
    const trimmed = emailStr.trim();
    if (trimmed.toLowerCase() === 'admin') {
      return 'admin@admin.com';
    }
    return trimmed;
  };

  const isValidEmail = (emailStr) => {
    const processed = processEmailInput(emailStr);
    return /\S+@\S+\.\S+/.test(processed);
  };

  // NEIS 바이트 계산기
  const getByteLength = (str) => {
    if (!str) return 0;
    let byteLength = 0;
    for (let i = 0; i < str.length; i++) {
      const code = str.charCodeAt(i);
      if (code === 10) { 
        byteLength += 2; 
      } else if (code > 127) {
        byteLength += 3; 
      } else {
        byteLength += 1; 
      }
    }
    return byteLength;
  };

  // Firebase Auth 에러 메시지 매핑
  const getErrorMessage = (code) => {
    switch (code) {
      case 'auth/configuration-not-found': return '인증 설정 정보가 잘못되었거나 활성화되지 않았습니다.';
      case 'auth/email-already-in-use': return '이미 가입된 이메일 주소입니다.';
      case 'auth/invalid-email': return '이메일 형식에 맞지 않습니다.';
      case 'auth/weak-password': return '비밀번호는 최소 6자 이상으로 설정해 주세요.';
      case 'auth/invalid-credential': return '이메일 주소 또는 비밀번호가 올바르지 않습니다.';
      case 'auth/user-not-found': return '등록되지 않은 회원정보입니다.';
      case 'auth/wrong-password': return '비밀번호가 일치하지 않습니다.';
      case 'auth/too-many-requests': return '요청이 집중되어 임시 차단되었습니다. 잠시 후 다시 시도해 주세요.';
      default: return `연결 중 오류가 발생했습니다 (${code})`;
    }
  };

  // 커스텀 모달 호출기
  const showConfirm = (title, message, onConfirm) => {
    setModal({
      show: true,
      title,
      message,
      onConfirm: () => {
        onConfirm();
        setModal({ show: false, title: '', message: '', onConfirm: null });
      }
    });
  };

  // 초기 로그인 상태 확인 및 세션 검사
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        }
      } catch (err) {
        console.warn("인증 토큰 로그인 실패, 익명 로그인 또는 수동 로그인으로 대체합니다:", err);
        try {
          await signInAnonymously(auth);
        } catch (anonErr) {
          console.error("익명 로그인 대체 실패:", anonErr);
        }
      } finally {
        setLoading(false);
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setLoading(false);
      if (currentUser) {
        setView('main');
        setMessage({ type: '', text: '' });
      } else {
        setView('login');
      }
    });
    return () => unsubscribe();
  }, []);

  // Firestore 실시간 데이터 로드 (1, 2, 3학년 모두 구독 및 기본 구조 채우기)
  useEffect(() => {
    if (!user) return;

    const unsubscribes = ['1', '2', '3'].map((g) => {
      const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'records', `grade_${g}`);
      
      return onSnapshot(docRef, (docSnap) => {
        if (docSnap.exists()) {
          const data = docSnap.data();
          setAllGradesData(prev => ({
            ...prev,
            [g]: {
              fixed: data.fixed || {},
              subjects: data.subjects || [],
              grades: data.grades || {},
              targetDept: data.targetDept || '컴퓨터공학과'
            }
          }));
          if (g === grade) {
            setSelectedDept(data.targetDept || '컴퓨터공학과');
          }
        } else {
          // 기록 데이터가 존재하지 않을 때, 기본 필수 교과목 구성 반영
          const initialFixed = {};
          Object.keys(FIXED_STRUCTURE).forEach(key => initialFixed[key] = "");

          const defaultSubjectsForGrade = DEFAULT_GRADE_SUBJECTS[g].map((sub, index) => ({
            id: `default_${g}_${index}_${Date.now()}`,
            name: sub.name,
            content: sub.content,
            limit: sub.limit
          }));

          setAllGradesData(prev => ({
            ...prev,
            [g]: {
              fixed: initialFixed,
              subjects: defaultSubjectsForGrade,
              grades: {},
              targetDept: '컴퓨터공학과'
            }
          }));
          if (g === grade) {
            setSelectedDept('컴퓨터공학과');
          }
        }
      }, (error) => {
        console.error(`${g}학년 데이터 실시간 동기화 실패:`, error);
        setSaveStatus('sync-error');
      });
    });

    return () => unsubscribes.forEach(unsub => unsub());
  }, [user]);

  // 학년 변경 시 해당 학년의 선택 학과 가져오기
  useEffect(() => {
    if (allGradesData[grade]) {
      setSelectedDept(allGradesData[grade].targetDept || '컴퓨터공학과');
      setAiFeedbackText(''); // 학년 변경 시 이전 학년 피드백 텍스트 초기화
    }
  }, [grade]);

  // 회원가입, 로그인, 재설정 통합 처리기
  const handleAuthAction = async (e, actionType) => {
    e.preventDefault();
    setMessage({ type: '', text: '' });

    const finalEmail = processEmailInput(email);

    if (!isValidEmail(finalEmail)) {
      setMessage({ type: 'error', text: '정확한 이메일 형식을 기입해 주세요.' });
      return;
    }

    try {
      if (actionType === 'signup') {
        if (password.length < 6) {
          setMessage({ type: 'error', text: '비밀번호는 최소 6자 이상이어야 합니다.' });
          return;
        }
        await createUserWithEmailAndPassword(auth, finalEmail, password);
        setMessage({ type: 'success', text: '성공적으로 가입되었습니다! 로그인 중입니다...' });
      } else if (actionType === 'login') {
        await signInWithEmailAndPassword(auth, finalEmail, password);
      } else if (actionType === 'reset') {
        await sendPasswordResetEmail(auth, finalEmail);
        setMessage({ type: 'success', text: '비밀번호 재설정 메일이 발송되었습니다. 메일함을 확인해 주세요.' });
      }
    } catch (err) {
      if (actionType === 'login' && (err.code === 'auth/user-not-found' || err.code === 'auth/invalid-credential') && (email.trim() === 'admin' || email.trim() === 'admin@admin.com')) {
        try {
          setMessage({ type: 'success', text: '관리자 데모 계정을 신규 생성하고 있습니다...' });
          await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
          await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
          return;
        } catch (signUpErr) {
          console.error("데모계정 생성 실패", signUpErr);
        }
      }
      setMessage({ type: 'error', text: getErrorMessage(err.code) });
    }
  };

  // 데모 어드민 로그인
  const handleDemoLogin = async () => {
    setMessage({ type: '', text: '' });
    setEmail('admin');
    setPassword('admin09');
    
    try {
      await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
    } catch (err) {
      if (err.code === 'auth/user-not-found' || err.code === 'auth/invalid-credential') {
        try {
          await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
          await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
        } catch (createErr) {
          setMessage({ type: 'error', text: '데모 계정 생성 중 오류 발생: ' + createErr.message });
        }
      } else {
        setMessage({ type: 'error', text: getErrorMessage(err.code) });
      }
    }
  };

  // 확실하게 세션을 종료하고 강제 로그인 뷰로 회귀하는 로그아웃 핸들러
  const handleLogout = async () => {
    try {
      // 로컬 데이터를 즉시 청소하여 UI 렌더링에 반응하도록 보장합니다.
      setUser(null);
      setAllGradesData({
        '1': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
        '2': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
        '3': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }
      });
      setEmail('');
      setPassword('');
      setView('login'); // UI 화면 상태 즉시 로그인 페이지로 강제 복귀
      setMessage({ type: 'success', text: '안전하게 로그아웃 되었습니다.' });
      
      // Firebase 상에서 세션을 완전히 파괴합니다.
      await signOut(auth);
    } catch (err) {
      console.error("비동기 로그아웃 실패(수동 세션 초기화 완료):", err);
      // Firebase 라이브러리 오류가 있더라도 사용자는 확실히 튕겨냅니다.
      setUser(null);
      setView('login');
    }
  };

  // 고정 필드 텍스트 변경
  const updateFixedField = (field, value) => {
    const limit = FIXED_STRUCTURE[field].limit;
    const currentFixed = allGradesData[grade]?.fixed || {};
    const updatedValue = value.length > limit ? value.substring(0, limit) : value;
    
    setAllGradesData(prev => ({
      ...prev,
      [grade]: {
        ...prev[grade],
        fixed: {
          ...currentFixed,
          [field]: updatedValue
        }
      }
    }));
  };

  // 과목 선택 후 최종 추가 처리
  const handleAddSelectedSubjectName = (subjectName) => {
    const currentSubjects = allGradesData[grade]?.subjects || [];

    if (currentSubjects.some(sub => sub.name === subjectName)) {
      setMessage({ type: 'error', text: `이미 '${subjectName}' 과목이 리스트에 존재합니다.` });
      setTimeout(() => setMessage({ type: '', text: '' }), 3000);
      setIsAddingSubject(false);
      return;
    }

    const newSub = {
      id: `custom_${Date.now()}`,
      name: subjectName,
      content: '',
      limit: 500 
    };

    setAllGradesData(prev => ({
      ...prev,
      [grade]: {
        ...prev[grade],
        subjects: [...currentSubjects, newSub]
      }
    }));

    setIsAddingSubject(false);
    setMessage({ type: '', text: '' });
  };

  // 과목 직접 타이핑 추가
  const handleAddManualSubject = () => {
    const trimmedName = newSubjectName.trim();
    if (!trimmedName) return;
    handleAddSelectedSubjectName(trimmedName);
    setNewSubjectName('');
  };

  // 과목 삭제
  const handleDeleteSubject = (id, name) => {
    showConfirm(
      '과목 삭제 확인',
      `정말 '${name}' 과목 기록을 삭제하시겠습니까? 삭제한 정보는 되돌릴 수 없습니다.`,
      () => {
        const currentSubjects = allGradesData[grade]?.subjects || [];
        const currentGrades = allGradesData[grade]?.grades || {};
        const updatedGrades = { ...currentGrades };
        
        // 1학년 공통과목 분할 삭제 대응
        if (grade === '1') {
          delete updatedGrades[`${id}_1`];
          delete updatedGrades[`${id}_2`];
        }
        delete updatedGrades[id];

        setAllGradesData(prev => ({
          ...prev,
          [grade]: {
            ...prev[grade],
            subjects: currentSubjects.filter(s => s.id !== id),
            grades: updatedGrades
          }
        }));
      }
    );
  };

  // 과목 세특 입력값 변경
  const updateSubjectContent = (id, value) => {
    const currentSubjects = allGradesData[grade]?.subjects || [];
    const updatedSubjects = currentSubjects.map(s => {
      if (s.id === id) {
        return {
          ...s,
          content: value.length > s.limit ? value.substring(0, s.limit) : value
        };
      }
      return s;
    });

    setAllGradesData(prev => ({
      ...prev,
      [grade]: {
        ...prev[grade],
        subjects: updatedSubjects
      }
    }));
  };

  // 성적 데이터 업데이트 (1학기, 2학기 각각 세분화 처리)
  const updateGradeData = (subId, field, value) => {
    const currentGrades = allGradesData[grade]?.grades || {};
    const updatedGrades = {
      ...currentGrades,
      [subId]: {
        ...currentGrades[subId],
        [field]: Number(value)
      }
    };

    setAllGradesData(prev => ({
      ...prev,
      [grade]: {
        ...prev[grade],
        grades: updatedGrades
      }
    }));
  };

  // 학과 선택 및 상태 저장
  const handleSelectDepartment = (deptName) => {
    setSelectedDept(deptName);
    setIsDeptDropdownOpen(false);
    setDeptSearch('');

    setAllGradesData(prev => ({
      ...prev,
      [grade]: {
        ...prev[grade],
        targetDept: deptName
      }
    }));
  };

  // Firestore 실시간 클라우드 데이터 전송 및 백업
  const saveToCloud = async () => {
    if (!user) return;
    setSaveStatus('saving');
    try {
      const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'records', `grade_${grade}`);
      const dataToSave = allGradesData[grade];
      await setDoc(docRef, {
        fixed: dataToSave.fixed || {},
        subjects: dataToSave.subjects || [],
        grades: dataToSave.grades || {},
        targetDept: selectedDept,
        lastUpdated: new Date().toISOString()
      });
      setSaveStatus('success');
      setTimeout(() => setSaveStatus(''), 2500);
    } catch (err) {
      console.error("기록 백업 저장 실패:", err);
      setSaveStatus('error');
      setTimeout(() => setSaveStatus(''), 3000);
    }
  };

  // 교양과목/미산출 과목 여부 체크 판별식
  const isLiberalArtSubject = (name) => {
    if (!name) return false;
    return EXCLUDED_LIBERAL_ARTS.some(artKeyword => name.toLowerCase().includes(artKeyword));
  };

  // 1학년 공통과목 학기 분리 리스트 빌더
  const getGradesTabSubjects = () => {
    const activeSubjects = allGradesData[grade]?.subjects || [];
    
    if (grade !== '1') return activeSubjects;

    // 1학년인 경우 성적 분석 탭 한정으로 공통영어, 공통수학, 공통국어, 통합사회, 통합과학을 1과 2로 분할
    const splitSubjects = [];
    activeSubjects.forEach(sub => {
      const targetNamesToSplit = ["공통국어", "공통수학", "공통영어", "통합사회", "통합과학"];
      if (targetNamesToSplit.includes(sub.name)) {
        splitSubjects.push({
          id: `${sub.id}_1`,
          name: `${sub.name}1`,
          limit: sub.limit,
          isSplit: true
        });
        splitSubjects.push({
          id: `${sub.id}_2`,
          name: `${sub.name}2`,
          limit: sub.limit,
          isSplit: true
        });
      } else {
        splitSubjects.push(sub);
      }
    });

    return splitSubjects;
  };

  // 학년별 GPA 세부 통계 연산
  const calculateGPAByGrade = (targetGrade) => {
    const currentGrades = allGradesData[targetGrade]?.grades || {};
    const activeSubjects = allGradesData[targetGrade]?.subjects || [];

    // 대상 학년이 1학년일 때 수식용 리스트 재조정
    const processingSubjects = [];
    if (targetGrade === '1') {
      activeSubjects.forEach(sub => {
        const targetNamesToSplit = ["공통국어", "공통수학", "공통영어", "통합사회", "통합과학"];
        if (targetNamesToSplit.includes(sub.name)) {
          processingSubjects.push({ id: `${sub.id}_1`, name: `${sub.name}1` });
          processingSubjects.push({ id: `${sub.id}_2`, name: `${sub.name}2` });
        } else {
          processingSubjects.push(sub);
        }
      });
    } else {
      processingSubjects.push(...activeSubjects);
    }

    let totalUnits9 = 0;
    let sumGrades9 = 0;
    let totalUnits5 = 0;
    let sumGrades5 = 0;

    processingSubjects.forEach(sub => {
      if (!isLiberalArtSubject(sub.name)) {
        const subGrade = currentGrades[sub.id] || { units: 0, grade9: 0, grade5: 0 };
        const units = subGrade.units || 0;
        const g9 = subGrade.grade9 || 0;
        const g5 = subGrade.grade5 || 0;

        if (units > 0) {
          if (g9 > 0) {
            totalUnits9 += units;
            sumGrades9 += (g9 * units);
          }
          if (g5 > 0) {
            totalUnits5 += units;
            sumGrades5 += (g5 * units);
          }
        }
      }
    });

    const average9 = totalUnits9 > 0 ? (sumGrades9 / totalUnits9) : null;
    const average5 = totalUnits5 > 0 ? (sumGrades5 / totalUnits5) : null;

    return { average9, average5 };
  };

  // 성적 추세 분석 엔진 (우상향, 우하향 등 판정)
  const analyzeTrend = (gradePoints) => {
    const validPoints = gradePoints.filter(p => p.val !== null);
    if (validPoints.length < 2) {
      return { text: "분석을 위한 성적 데이터가 부족합니다.", icon: "neutral" };
    }

    let first = validPoints[0];
    let last = validPoints[validPoints.length - 1];

    const diff = first.val - last.val; 

    if (diff > 0.15) {
      return { 
        text: "학년이 갈수록 등급이 낮아지는(성적이 상승하는) 대표적인 '지속 우상향(학업 성취도 개선형)' 흐름입니다. 대입 학생부 종합전형에서 학업 주도성과 발전 가능성 부문에서 매우 긍정적인 정성 평가를 받을 확률이 높습니다.", 
        icon: "up" 
      };
    } else if (diff < -0.15) {
      return { 
        text: "학년이 올라갈수록 등급 수치가 올라가는 '우하향(학업 관리 보완 필요형)' 흐름이 감지되었습니다. 3학년 1학기 최종 내신 및 주력 선택과목들의 성적 방어가 필요하며, 학생부 기재 내용(세특)을 통해 학업적 성실성을 추가 보강하는 보완 전략이 요구됩니다.", 
        icon: "down" 
      };
    } else {
      return { 
        text: "일정한 내신 등급 구간을 유지하는 '성적 안정형(지속 유지형)' 흐름입니다. 급격한 편차 없이 성실함을 꾸준히 증명하고 있으므로, 전공 핵심 권장 과목의 이수 여부와 성취도 수준이 입시 성공의 핵심 열쇠가 될 것입니다.", 
        icon: "stable" 
      };
    }
  };

  // 실시간 Canvas 내신 성적 그래프 드로잉 이펙트
  useEffect(() => {
    if (activeTab !== 'grades' || !canvasRef.current) return;

    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    canvas.width = 600;
    canvas.height = 300;

    const data1 = calculateGPAByGrade('1');
    const data2 = calculateGPAByGrade('2');
    const data3 = calculateGPAByGrade('3');

    const points9 = [
      { label: '고1', val: data1.average9 },
      { label: '고2', val: data2.average9 },
      { label: '고3', val: data3.average9 }
    ];

    const points5 = [
      { label: '고1', val: data1.average5 },
      { label: '고2', val: data2.average5 },
      { label: '고3', val: data3.average5 }
    ];

    // 배경 지우기
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 모눈 눈금 그리기
    ctx.strokeStyle = isDarkMode ? '#1e293b' : '#f1f5f9';
    ctx.lineWidth = 1;
    for (let i = 1; i < 9; i++) {
      const y = 30 + i * 25;
      ctx.beginPath();
      ctx.moveTo(60, y);
      ctx.lineTo(550, y);
      ctx.stroke();

      ctx.fillStyle = isDarkMode ? '#94a3b8' : '#64748b';
      ctx.font = 'bold 10px sans-serif';
      ctx.fillText(`${i}등급`, 25, y + 3);
    }

    const xCoords = [120, 300, 480];
    ctx.strokeStyle = isDarkMode ? '#334155' : '#cbd5e1';
    ctx.beginPath();
    ctx.moveTo(60, 255);
    ctx.lineTo(550, 255);
    ctx.stroke();

    points9.forEach((p, idx) => {
      ctx.fillStyle = isDarkMode ? '#cbd5e1' : '#334155';
      ctx.font = 'bold 12px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(p.label, xCoords[idx], 275);
    });

    drawTrendLine(ctx, points9, xCoords, '#3b82f6', '#60a5fa');
    drawTrendLine(ctx, points5, xCoords, '#8b5cf6', '#a78bfa');

  }, [activeTab, allGradesData, isDarkMode]);

  const drawTrendLine = (ctx, points, xCoords, strokeColor, dotColor) => {
    const validCoords = [];
    points.forEach((p, idx) => {
      if (p.val !== null) {
        const y = 30 + (p.val - 1) * 25;
        validCoords.push({ x: xCoords[idx], y, val: p.val });
      }
    });

    if (validCoords.length === 0) return;

    ctx.strokeStyle = strokeColor;
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.moveTo(validCoords[0].x, validCoords[0].y);
    for (let i = 1; i < validCoords.length; i++) {
      ctx.lineTo(validCoords[i].x, validCoords[i].y);
    }
    ctx.stroke();

    validCoords.forEach(pt => {
      ctx.fillStyle = dotColor;
      ctx.beginPath();
      ctx.arc(pt.x, pt.y, 6, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.arc(pt.x, pt.y, 2.5, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = isDarkMode ? '#f8fafc' : '#0f172a';
      ctx.font = 'black 11px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(`${pt.val.toFixed(2)}`, pt.x, pt.y - 12);
    });
  };

  // GPA 요약 통계 계산
  const calculateGPA = () => {
    const currentGrades = allGradesData[grade]?.grades || {};
    const activeSubjects = getGradesTabSubjects();

    let totalUnits9 = 0;
    let sumGrades9 = 0;
    
    let totalUnits5 = 0;
    let sumGrades5 = 0;

    activeSubjects.forEach(sub => {
      if (!isLiberalArtSubject(sub.name)) {
        const subGrade = currentGrades[sub.id] || { units: 0, grade9: 0, grade5: 0 };
        const units = subGrade.units || 0;
        const g9 = subGrade.grade9 || 0;
        const g5 = subGrade.grade5 || 0;

        if (units > 0) {
          if (g9 > 0) {
            totalUnits9 += units;
            sumGrades9 += (g9 * units);
          }
          if (g5 > 0) {
            totalUnits5 += units;
            sumGrades5 += (g5 * units);
          }
        }
      }
    });

    const average9 = totalUnits9 > 0 ? (sumGrades9 / totalUnits9).toFixed(2) : '-';
    const average5 = totalUnits5 > 0 ? (sumGrades5 / totalUnits5).toFixed(2) : '-';

    return { average9, average5, totalUnits9, totalUnits5 };
  };

  const { average9, average5, totalUnits9, totalUnits5 } = calculateGPA();

  // 1학년부터 3학년까지 통합 성적 데이터 수집
  const gradePoints9 = [
    { label: '고1', val: calculateGPAByGrade('1').average9 },
    { label: '고2', val: calculateGPAByGrade('2').average9 },
    { label: '고3', val: calculateGPAByGrade('3').average9 }
  ];
  const trendAnalysis = analyzeTrend(gradePoints9);

  // ==========================================================
  // [GEMINI API INTEGRATION FOR AI FEEDBACK]
  // ==========================================================
  const callGeminiFeedbackAPI = async (userQuery, systemPrompt) => {
    const apiKey = ""; 
    let attempts = 0;
    const maxRetries = 5;

    while (attempts < maxRetries) {
      try {
        const response = await fetch(
          `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`,
          {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              contents: [{ parts: [{ text: userQuery }] }],
              systemInstruction: { parts: [{ text: systemPrompt }] }
            })
          }
        );

        if (!response.ok) {
          throw new Error(`HTTP Error Status: ${response.status}`);
        }

        const result = await response.json();
        const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
        if (text) return text;
        throw new Error("Invalid Response Body Structure");
      } catch (err) {
        attempts++;
        if (attempts >= maxRetries) {
          throw err;
        }
        const delay = Math.pow(2, attempts - 1) * 1000;
        await new Promise(res => setTimeout(res, delay));
      }
    }
  };

  // AI 분석 피드백 실행 핸들러
  const handleGenerateAIFeedback = async () => {
    setIsGeneratingFeedback(true);
    setAiFeedbackText('');

    const targetDeptName = selectedDept;
    const activeSubjects = allGradesData[grade]?.subjects || [];
    const fixedActivities = allGradesData[grade]?.fixed || {};

    let recordSummary = `[목표 학과] ${targetDeptName}\n\n`;
    recordSummary += `[교과 학습 발달 상황 (세특)]\n`;
    if (activeSubjects.length === 0) {
      recordSummary += `- 등록된 교과 세특 내용 없음\n`;
    } else {
      activeSubjects.forEach(s => {
        recordSummary += `- 과목명: ${s.name}\n  세특내용: ${s.content || '작성 전'}\n\n`;
      });
    }

    recordSummary += `[창의적 체험활동]\n`;
    recordSummary += `- 자율활동: ${fixedActivities.autonomous || '작성 전'}\n`;
    recordSummary += `- 동아리활동: ${fixedActivities.club || '작성 전'}\n`;
    recordSummary += `- 진로활동: ${fixedActivities.career || '작성 전'}\n`;
    recordSummary += `- 행동특성 및 종합의견: ${fixedActivities.comprehensive || '작성 전'}\n`;

    const systemPrompt = `
    당신은 고등학교 생활기록부 분석 및 대입 컨설팅 전문가입니다. 
    2022 개정 교육과정 및 고교학점제 평가 요소에 기반하여, 학생의 현재 작성 내역이 목표 학과인 [${targetDeptName}]에 얼마나 부합하는지 정밀 피드백을 작성해 주세요.
    
    작성 지침:
    1. 어조는 학생에게 따뜻하게 조언해 주는 존댓말 기조로 작성해 주세요.
    2. 마크다운 형식을 적극 활용하여 가독성 있게 구조화해 주세요.
    3. 아래 4가지 항목을 필수로 명확하게 진단해 주세요:
       - 🌟 [종합 평가 및 전공적합성 수준]
       - 🔍 [작성된 초안의 명확한 부족한 점 및 아쉬운 부분]
       - 💡 [2022 개정 대입에 유리하도록 수정·보완해야 할 핵심 구체적 방향 가이드]
       - 🔬 [목표 학과 학업 역량을 강화할 수 있는 추천 탐구 키워드 및 학술 독서 추천]
    `;

    try {
      const resultText = await callGeminiFeedbackAPI(recordSummary, systemPrompt);
      setAiFeedbackText(resultText);
    } catch (err) {
      console.error("Gemini AI Feedback API Call Failed", err);
      setAiFeedbackText("⚠️ AI 서버 연결 지연 혹은 일시적인 오류가 발생했습니다. 잠시 후 [AI 분석 리포트 생성하기] 버튼을 다시 클릭해 주세요.");
    } finally {
      setIsGeneratingFeedback(false);
    }
  };

  // 학과 리스트 검색 필터링
  const filteredDepartments = UNIVERSITY_DEPARTMENTS.filter(dept =>
    dept.toLowerCase().includes(deptSearch.toLowerCase())
  );

  const currentGradeData = allGradesData[grade] || { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' };
  const records = currentGradeData.fixed || {};
  const subjects = currentGradeData.subjects || [];
  const grades = currentGradeData.grades || {};

  // 성적 입력용으로 가공된 과목 리스트 생성
  const gradesTabSubjects = getGradesTabSubjects();

  return (
    <div className={`min-h-screen font-sans pb-24 transition-colors duration-300 ${isDarkMode ? 'bg-slate-950 text-slate-100' : 'bg-slate-50 text-slate-800'}`}>
      
      {/* 헤더 바 */}
      <header className={`sticky top-0 z-20 border-b backdrop-blur-md transition-colors ${isDarkMode ? 'bg-slate-900/80 border-slate-800/80' : 'bg-white/80 border-slate-200/50'}`}>
        <div className="max-w-5xl mx-auto px-4 h-16 flex items-center justify-between">
          <div className="flex items-center gap-2.5">
            <div className="w-9 h-9 bg-gradient-to-tr from-blue-600 to-indigo-600 rounded-xl flex items-center justify-center text-white shadow-sm">
              <BookOpen className="w-5 h-5" />
            </div>
            <div className="flex flex-col">
              <span className="font-extrabold text-sm tracking-tight">생기부 매니저</span>
              <span className={`text-[10px] font-bold hidden sm:block ${isDarkMode ? 'text-slate-500' : 'text-slate-400'}`}>고교학점제 & 2022 개정 맞춤</span>
            </div>
          </div>

          {/* 학년 고1, 2, 3 선택 버튼 */}
          <div className={`flex items-center p-1.5 rounded-xl border ${isDarkMode ? 'bg-slate-800 border-slate-700/50' : 'bg-slate-100 border-slate-200/20'}`}>
            {['1', '2', '3'].map((g) => (
              <button 
                key={g} 
                onClick={() => setGrade(g)} 
                className={`px-4 py-1.5 rounded-lg text-xs font-black transition-all ${
                  grade === g 
                    ? (isDarkMode ? 'bg-slate-700 text-blue-400 shadow-sm' : 'bg-white text-blue-600 shadow-sm border border-slate-200/10')
                    : 'text-slate-400 hover:text-slate-350'
                }`}
              >
                고{g}
              </button>
            ))}
          </div>

          {/* 설정을 포함한 헤더 액션 */}
          <div className="flex items-center gap-2.5">
            <button 
              onClick={() => setIsSettingsOpen(true)}
              className={`p-2 rounded-xl transition-all ${isDarkMode ? 'text-slate-300 hover:bg-slate-800' : 'text-slate-500 hover:bg-slate-100'}`}
              title="설정 및 다크모드"
            >
              <Settings className="w-5 h-5 animate-hover-spin" />
            </button>

            <button 
              onClick={handleLogout} 
              className={`flex items-center gap-1.5 px-3 py-2 rounded-xl transition-all font-bold text-xs ${
                isDarkMode ? 'text-slate-400 hover:text-red-400 hover:bg-red-950/20' : 'text-slate-400 hover:text-red-500 hover:bg-red-50'
              }`}
            >
              <LogOut className="w-4 h-4" />
              <span className="hidden md:inline">로그아웃</span>
            </button>
          </div>
        </div>
      </header>

      {/* 대시보드 기입 본문 */}
      <main className="max-w-4xl mx-auto px-4 pt-6">

        {/* 탭 네비게이션바 (기록장 / 성적 분석 / AI 피드백) */}
        <div className="mb-8 flex justify-center">
          <div className={`flex p-1.5 rounded-2xl w-full max-w-xl shadow-sm border ${
            isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200/60'
          }`}>
            <button
              onClick={() => setActiveTab('record')}
              className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                activeTab === 'record'
                  ? 'bg-blue-600 text-white shadow-md'
                  : (isDarkMode ? 'text-slate-400 hover:text-slate-200 hover:bg-slate-800/45' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50')
              }`}
            >
              <FileText className="w-4 h-4" />
              기록장
            </button>
            <button
              onClick={() => setActiveTab('grades')}
              className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                activeTab === 'grades'
                  ? 'bg-blue-600 text-white shadow-md'
                  : (isDarkMode ? 'text-slate-400 hover:text-slate-200 hover:bg-slate-800/45' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50')
              }`}
            >
              <BarChart3 className="w-4 h-4" />
              성적 분석
            </button>
            <button
              onClick={() => setActiveTab('feedback')}
              className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                activeTab === 'feedback'
                  ? 'bg-blue-600 text-white shadow-md'
                  : (isDarkMode ? 'text-slate-400 hover:text-slate-200 hover:bg-slate-800/45' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50')
              }`}
            >
              <BrainCircuit className="w-4 h-4" />
              AI 피드백
            </button>
          </div>
        </div>
        
        {/* 상단 안내 패널 */}
        <div className="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-6">
          <div className="space-y-2">
            <div className={`inline-flex items-center gap-1.5 px-2.5 py-1 border rounded-full text-[10px] font-black ${
              isDarkMode 
                ? 'bg-blue-950/30 border-blue-900/50 text-blue-400' 
                : 'bg-gradient-to-r from-blue-50 to-indigo-50 border border-blue-100/50 text-blue-600'
            }`}>
              <Sparkles className="w-3 h-3 text-blue-500 animate-pulse" />
              2022 개정 교육과정 규칙 반영됨
            </div>
            <h2 className="text-2xl md:text-3xl font-extrabold tracking-tight">
              고등학교 {grade}학년 {activeTab === 'record' ? '생기부 기록장' : activeTab === 'grades' ? '교과 성적 관리기' : 'AI 피드백 가이드'}
            </h2>
            <p className={`text-xs font-semibold leading-relaxed ${isDarkMode ? 'text-slate-400' : 'text-slate-455'}`}>
              {activeTab === 'record' && '학년별 교과세특과 창의적 체험활동을 체계적으로 백업하고 바이트를 미리 계산해보세요.'}
              {activeTab === 'grades' && '과목별 단위수와 석차등급을 기반으로 본인의 이수 단위 종합 등급을 산출해 보세요.'}
              {activeTab === 'feedback' && '목표 대학 학과를 지정하고, Gemini AI 평가 엔진으로 생기부 수시 경쟁력을 정량·정성 피드백 받아 보세요.'}
            </p>
          </div>
          
          {/* 변경 내역 저장 버튼 */}
          <div className="flex items-center gap-3">
            {saveStatus === 'saving' && (
              <span className="text-[11px] font-bold text-slate-400 flex items-center gap-1.5">
                <RefreshCw className="w-3 h-3 animate-spin text-blue-500" />
                보안 서버에 실시간 기록 중...
              </span>
            )}
            {saveStatus === 'success' && (
              <span className="text-[11px] font-extrabold text-green-600 bg-green-50 dark:bg-green-950/30 px-3 py-1.5 rounded-xl border border-green-100 dark:border-green-900/50 flex items-center gap-1">
                <CheckCircle2 className="w-3.5 h-3.5" />
                성공적으로 백업 저장 완료!
              </span>
            )}
            {saveStatus === 'error' && (
              <span className="text-[11px] font-extrabold text-red-600 bg-red-50 dark:bg-red-950/30 px-3 py-1.5 rounded-xl border border-red-100 dark:border-red-900/50 flex items-center gap-1">
                <AlertCircle className="w-3.5 h-3.5" />
                인터넷 임시 지연 (재시도 중)
              </span>
            )}
            <button 
              onClick={saveToCloud} 
              className="flex items-center gap-2 px-6 py-3.5 bg-blue-600 hover:bg-blue-700 text-white rounded-2xl font-black shadow-lg shadow-blue-100 dark:shadow-none transition-all active:scale-95 text-xs md:text-sm"
            >
              <Save className="w-4.5 h-4.5" />
              클라우드 영구 저장
            </button>
          </div>
        </div>

        {/* ==========================================
            [TAB 1: RECORD - 기록장 화면]
           ========================================== */}
        {activeTab === 'record' && (
          <>
            {/* 과목 세특 섹션 */}
            <section className="mb-14 animate-in fade-in slide-in-from-bottom-3 duration-300">
              <div className="flex items-center justify-between mb-5">
                <h3 className="text-lg font-black flex items-center gap-2">
                  <span className="w-1.5 h-5 bg-gradient-to-b from-blue-500 to-indigo-600 rounded-full inline-block" />
                  교과 학습 발달 상황 (과목별 세특)
                </h3>
                
                {/* 1학년이 아닐 때만 과목 추가 버튼 노출 */}
                {grade !== '1' && (
                  <button 
                    onClick={() => setIsAddingSubject(true)}
                    className={`flex items-center gap-1 px-3 py-1.5 rounded-lg font-extrabold text-xs transition-all ${
                      isDarkMode 
                        ? 'bg-slate-800 hover:bg-slate-700 text-blue-400' 
                        : 'bg-blue-50/80 hover:bg-blue-100 text-blue-600'
                    }`}
                  >
                    <Plus className="w-3.5 h-3.5" />
                    과목 추가 및 선택
                  </button>
                )}
              </div>

              {/* 과목 추가 및 선택 모달/인터페이스 (고1일 때는 진입 원천 차단) */}
              {isAddingSubject && grade !== '1' && (
                <div className={`mb-6 p-6 border rounded-2xl shadow-sm animate-in fade-in zoom-in-95 space-y-5 ${
                  isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-blue-100'
                }`}>
                  {/* 학년별 선택 리스트 제공 */}
                  {SELECTABLE_OPTIONAL_SUBJECTS[grade] && SELECTABLE_OPTIONAL_SUBJECTS[grade].length > 0 && (
                    <div>
                      <span className="block text-xs font-black text-slate-400 mb-2.5 tracking-wide">
                        고등학교 {grade}학년 추천 선택과목 리스트 (클릭하여 기입장에 자동 추가)
                      </span>
                      <div className="flex flex-wrap gap-2 max-h-48 overflow-y-auto p-1 border border-slate-100 dark:border-slate-800/80 rounded-xl bg-slate-50/40 dark:bg-slate-950/20">
                        {SELECTABLE_OPTIONAL_SUBJECTS[grade].map((subName, index) => {
                          const isAlreadyAdded = subjects.some(s => s.name === subName);
                          return (
                            <button
                              key={index}
                              onClick={() => handleSelectDepartment && handleAddSelectedSubjectName(subName)}
                              disabled={isAlreadyAdded}
                              className={`px-3 py-1.5 text-xs font-bold rounded-lg border transition-all ${
                                isAlreadyAdded 
                                  ? 'bg-slate-100 dark:bg-slate-850 text-slate-350 dark:text-slate-650 border-transparent cursor-not-allowed'
                                  : 'bg-white dark:bg-slate-850 hover:bg-blue-50 dark:hover:bg-blue-950/30 text-slate-700 dark:text-slate-300 border-slate-200 dark:border-slate-700 hover:border-blue-300'
                              }`}
                            >
                              {subName} {isAlreadyAdded && '(추가됨)'}
                            </button>
                          );
                        })}
                      </div>
                    </div>
                  )}

                  {/* 수동 커스텀 과목명 추가 공간 */}
                  <div className="pt-2 border-t border-slate-100 dark:border-slate-800/50">
                    <label className="block text-[10px] font-extrabold text-slate-400 mb-2 tracking-wider">직접 입력하여 수동 추가</label>
                    <div className="flex gap-2">
                      <input 
                        type="text" 
                        value={newSubjectName}
                        onChange={e => setNewSubjectName(e.target.value)}
                        onKeyDown={e => e.key === 'Enter' && handleAddManualSubject()}
                        placeholder="예: 고급 수학, 창의 융합 과학"
                        className={`flex-1 px-4 py-2.5 border rounded-xl focus:ring-2 focus:ring-blue-500 font-semibold text-xs outline-none ${
                          isDarkMode 
                            ? 'bg-slate-800 border-slate-700 text-white placeholder:text-slate-600' 
                            : 'bg-slate-50 border-slate-100 text-slate-900 placeholder:text-slate-300'
                        }`}
                        maxLength={25}
                      />
                      <button onClick={handleAddManualSubject} className="px-5 bg-blue-600 text-white font-bold rounded-xl hover:bg-blue-700 transition-colors text-xs">등록</button>
                      <button onClick={() => { setIsAddingSubject(false); setNewSubjectName(''); }} className={`px-3 font-bold rounded-xl transition-all text-xs ${isDarkMode ? 'bg-slate-850 text-slate-450 hover:bg-slate-800' : 'bg-slate-100 text-slate-400 hover:bg-slate-200'}`}>취소</button>
                    </div>
                  </div>
                </div>
              )}

              {/* 과목 세특 카드 목록 */}
              <div className="grid gap-6">
                {subjects.length === 0 && !isAddingSubject && (
                  <div className={`py-14 text-center rounded-2xl border border-dashed ${isDarkMode ? 'bg-slate-900/30 border-slate-850' : 'bg-white border-slate-200/80'}`}>
                    <FileText className="w-10 h-10 text-slate-200 dark:text-slate-800 mx-auto mb-3" />
                    <p className={`font-semibold text-xs leading-relaxed ${isDarkMode ? 'text-slate-500' : 'text-slate-400'}`}>
                      아직 등록된 교과 과목 세특란이 없습니다. <br/>
                      {grade !== '1' && '우측 상단 과목 추가 버튼을 클릭하여 과목을 선택하고 세특 초안 작성을 시작하세요.'}
                    </p>
                  </div>
                )}
                
                {subjects.map((sub) => {
                  const charCount = sub.content ? sub.content.length : 0;
                  const byteCount = getByteLength(sub.content);
                  const isWarning = byteCalcMode === 'char' ? charCount >= sub.limit * 0.9 : byteCount >= (sub.limit * 3) * 0.9;
                  
                  const currentCount = byteCalcMode === 'char' ? charCount : byteCount;
                  const maxLimit = byteCalcMode === 'char' ? sub.limit : (sub.limit * 3);
                  const isLiberal = isLiberalArtSubject(sub.name);

                  return (
                    <div key={sub.id} className={`rounded-2xl border shadow-sm overflow-hidden hover:shadow-md transition-all ${
                      isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100'
                    }`}>
                      <div className={`px-6 py-4 border-b flex items-center justify-between ${isDarkMode ? 'bg-slate-900/50 border-slate-800' : 'bg-slate-50 border-slate-100'}`}>
                        <div className="flex items-center gap-2">
                          <span className="w-2 h-2 rounded-full bg-blue-500" />
                          <span className="font-bold text-sm">{sub.name} 세부능력 및 특기사항</span>
                          {isLiberal && (
                            <span className="text-[9px] bg-amber-50 dark:bg-amber-950/20 text-amber-600 border border-amber-200/50 dark:border-amber-900/50 font-black px-1.5 py-0.5 rounded-md">
                              예체능/교양 (성적 미산출)
                            </span>
                          )}
                        </div>
                        <div className="flex items-center gap-3">
                          <div className={`text-[10px] font-black px-2 py-1 rounded-md border ${
                            isWarning 
                              ? 'bg-red-50 dark:bg-red-950/20 text-red-500 dark:text-red-400 border-red-100 dark:border-red-900/50 animate-pulse' 
                              : (isDarkMode ? 'bg-slate-800 text-slate-400 border-slate-700' : 'bg-white text-slate-400 border-slate-200/60')
                          }`}>
                            {currentCount} / {maxLimit}{byteCalcMode === 'char' ? '자' : 'Byte'}
                          </div>
                          {/* 고1이 아닐 때만 과목 삭제 활성화 */}
                          {grade !== '1' && (
                            <button 
                              onClick={() => handleDeleteSubject(sub.id, sub.name)} 
                              className="text-slate-300 dark:text-slate-650 hover:text-red-500 dark:hover:text-red-400 transition-colors p-1"
                              title="과목 삭제"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          )}
                        </div>
                      </div>
                      <div className="p-5">
                        <textarea
                          value={sub.content}
                          onChange={(e) => updateSubjectContent(sub.id, e.target.value)}
                          placeholder={`${sub.name} 수업에서 본인의 탐구, 발표, 조별과제 기여 내역을 기입해 보세요.`}
                          className={`w-full h-36 bg-transparent border-none resize-none leading-relaxed font-semibold text-sm outline-none focus:ring-0 ${
                            isDarkMode ? 'text-slate-200 placeholder:text-slate-700' : 'text-slate-700 placeholder:text-slate-300'
                          }`}
                        />
                      </div>
                    </div>
                  );
                })}
              </div>
            </section>

            {/* 창체 및 행동특성 종합의견 */}
            <section className="animate-in fade-in slide-in-from-bottom-3 duration-350">
              <h3 className="text-lg font-black mb-5 flex items-center gap-2">
                <span className="w-1.5 h-5 bg-gradient-to-b from-blue-500 to-indigo-600 rounded-full inline-block" />
                창의적 체험활동 및 행동특성 종합의견
              </h3>
              <div className="grid gap-6">
                {Object.entries(FIXED_STRUCTURE).map(([key, config]) => {
                  const value = records[key] || "";
                  const charCount = value.length;
                  const byteCount = getByteLength(value);
                  const isWarning = byteCalcMode === 'char' ? charCount >= config.limit * 0.9 : byteCount >= (config.limit * 3) * 0.9;
                  
                  const currentCount = byteCalcMode === 'char' ? charCount : byteCount;
                  const maxLimit = byteCalcMode === 'char' ? config.limit : (config.limit * 3);

                  return (
                    <div key={key} className={`rounded-2xl border p-6 shadow-sm hover:shadow-md transition-all ${
                      isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100'
                    }`}>
                      <div className="flex items-center justify-between mb-3.5">
                        <div className="flex flex-col">
                          <label className="text-sm font-extrabold">{config.label}</label>
                          <span className={`text-[10px] font-semibold mt-0.5 ${isDarkMode ? 'text-slate-500' : 'text-slate-400'}`}>{config.tip}</span>
                        </div>
                        <div className={`text-[10px] font-black px-2 py-1 rounded-md border ${
                          isWarning 
                            ? 'bg-red-50 dark:bg-red-950/20 text-red-600 border-red-100 dark:border-red-900/50' 
                            : (isDarkMode ? 'bg-slate-800 text-slate-400 border-slate-750' : 'bg-slate-50 text-slate-400 border-slate-100')
                        }`}>
                          {currentCount} / {maxLimit}{byteCalcMode === 'char' ? '자' : 'Byte'}
                        </div>
                      </div>
                      <textarea
                        value={value}
                        onChange={(e) => updateFixedField(key, e.target.value)}
                        placeholder={config.placeholder}
                        className={`w-full h-40 p-4 border rounded-xl focus:ring-2 focus:outline-none transition-all resize-none leading-relaxed font-semibold text-sm ${
                          isDarkMode 
                            ? 'bg-slate-850 border-slate-800 focus:ring-blue-900/50 focus:border-blue-800 text-slate-200 placeholder:text-slate-700' 
                            : 'bg-slate-50/50 border-slate-100 focus:ring-blue-100 focus:border-blue-300 focus:bg-white text-slate-700 placeholder:text-slate-300'
                        }`}
                      />
                      {isWarning && (
                        <p className="text-[11px] text-red-500 font-bold mt-2 flex items-center gap-1.5">
                          <AlertCircle className="w-3.5 h-3.5" />
                          글자 수 및 바이트 수 제한 기준에 가까워졌습니다. 내용을 다듬어 조절해 주세요.
                        </p>
                      )}
                    </div>
                  );
                })}
              </div>
            </section>
          </>
        )}

        {/* ==========================================
            [TAB 2: GRADES - 성적 분석 화면]
           ========================================== */}
        {activeTab === 'grades' && (
          <div className="space-y-8 animate-in fade-in slide-in-from-bottom-3 duration-300">
            
            {/* 총 내신 등급 산출 요약 대시보드 */}
            <div className={`grid grid-cols-1 md:grid-cols-2 gap-6 p-6 rounded-3xl border ${
              isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-gradient-to-tr from-blue-50/60 to-indigo-50/50 border-blue-100/50'
            }`}>
              {/* 9등급제 평균 카드 */}
              <div className={`p-6 rounded-2xl flex items-center justify-between ${
                isDarkMode ? 'bg-slate-950/50' : 'bg-white shadow-sm'
              }`}>
                <div className="space-y-1">
                  <span className={`text-[11px] font-extrabold flex items-center gap-1 ${isDarkMode ? 'text-slate-450' : 'text-slate-500'}`}>
                    <Award className="w-3.5 h-3.5 text-blue-500" />
                    현재 학년 종합 내신 등급 (9등급제)
                  </span>
                  <p className="text-4xl font-black tracking-tight text-blue-600">{average9} <span className="text-xs text-slate-400 font-bold">등급</span></p>
                  <p className="text-[10px] text-slate-400 font-bold">총 이수 단위수: {totalUnits9} 단위</p>
                </div>
                <div className={`w-12 h-12 rounded-full flex items-center justify-center ${isDarkMode ? 'bg-blue-950/30' : 'bg-blue-50'}`}>
                  <span className="text-lg font-black text-blue-600">9</span>
                </div>
              </div>

              {/* 5등급제 평균 카드 */}
              <div className={`p-6 rounded-2xl flex items-center justify-between ${
                isDarkMode ? 'bg-slate-950/50' : 'bg-white shadow-sm'
              }`}>
                <div className="space-y-1">
                  <span className={`text-[11px] font-extrabold flex items-center gap-1 ${isDarkMode ? 'text-slate-450' : 'text-slate-500'}`}>
                    <Award className="w-3.5 h-3.5 text-indigo-500" />
                    고교학점제 내신 변환 등급 (5등급제)
                  </span>
                  <p className="text-4xl font-black tracking-tight text-indigo-600">{average5} <span className="text-xs text-slate-400 font-bold">등급</span></p>
                  <p className="text-[10px] text-slate-400 font-bold">총 이수 단위수: {totalUnits5} 단위</p>
                </div>
                <div className={`w-12 h-12 rounded-full flex items-center justify-center ${isDarkMode ? 'bg-indigo-950/30' : 'bg-indigo-50'}`}>
                  <span className="text-lg font-black text-indigo-600">5</span>
                </div>
              </div>
            </div>

            {/* [성적 변화 분석 그래프 및 정성 평가 카드] */}
            <div className={`rounded-3xl border p-6 ${
              isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100 shadow-sm'
            }`}>
              <div className="mb-6">
                <h3 className="text-base font-extrabold flex items-center gap-2">
                  <Activity className="w-5 h-5 text-blue-500" />
                  실시간 연도별 등급 변화 추이 그래프 (1~3학년)
                </h3>
                <p className="text-[11px] text-slate-400 font-semibold mt-1">
                  학기별로 기입된 평균 등급 성적의 궤적을 렌더링하고 대입 전공역량 방향에 맞게 시각 분석합니다.
                </p>
              </div>

              <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 items-center">
                {/* 캔버스 영역 */}
                <div className="lg:col-span-2 flex justify-center bg-slate-50/50 dark:bg-slate-950/40 p-4 rounded-2xl border border-slate-100 dark:border-slate-800/60">
                  <canvas ref={canvasRef} className="max-w-full h-auto" />
                </div>

                {/* 정성 트렌드 분석 보고서 */}
                <div className={`p-5 rounded-2xl border h-full flex flex-col justify-center ${
                  trendAnalysis.icon === 'up' 
                    ? 'bg-green-500/5 border-green-200/50 dark:border-green-900/20' 
                    : trendAnalysis.icon === 'down' 
                    ? 'bg-red-500/5 border-red-200/50 dark:border-red-900/20' 
                    : 'bg-slate-500/5 border-slate-200/50 dark:border-slate-800/40'
                }`}>
                  <div className="flex items-center gap-2 mb-3">
                    {trendAnalysis.icon === 'up' ? (
                      <div className="w-8 h-8 rounded-full bg-green-100 dark:bg-green-950/40 flex items-center justify-center text-green-600">
                        <TrendingUp className="w-4.5 h-4.5" />
                      </div>
                    ) : trendAnalysis.icon === 'down' ? (
                      <div className="w-8 h-8 rounded-full bg-red-100 dark:bg-red-950/40 flex items-center justify-center text-red-600">
                        <TrendingDown className="w-4.5 h-4.5" />
                      </div>
                    ) : (
                      <div className="w-8 h-8 rounded-full bg-blue-100 dark:bg-blue-950/40 flex items-center justify-center text-blue-600">
                        <Activity className="w-4.5 h-4.5" />
                      </div>
                    )}
                    <span className="text-xs font-black text-slate-800 dark:text-slate-100">입시 등급 트렌드 리포트</span>
                  </div>
                  <p className="text-[11px] leading-relaxed text-slate-655 dark:text-slate-400 font-bold whitespace-pre-line">
                    {trendAnalysis.text}
                  </p>
                </div>
              </div>
            </div>

            {/* 과목별 개별 성적 입력 판넬 */}
            <div className={`rounded-3xl border p-6 ${
              isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100 shadow-sm'
            }`}>
              <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-6 pb-4 border-b border-slate-100 dark:border-slate-800">
                <div>
                  <h3 className="text-base font-extrabold flex items-center gap-2">
                    <BookMarked className="w-5 h-5 text-blue-500" />
                    교과별 세부 성적 기입
                  </h3>
                  <p className="text-[11px] text-slate-400 font-semibold mt-1">기록장 탭에서 등록한 과목들이 아래에 자동으로 연동됩니다.</p>
                </div>
                
                {/* 2022 개정 필터 안내 메시지 */}
                <div className="px-3 py-1.5 bg-amber-50 dark:bg-amber-950/20 rounded-xl border border-amber-100 dark:border-amber-900/30 text-[10px] text-amber-600 dark:text-amber-400 font-bold">
                  ⚠️ 체육·예술군 및 교양 과목은 등급 미산출 교과로 분류되어 성적표 입력란에서 자동 제외됩니다.
                </div>
              </div>

              {/* 과목 목록 및 입력 표 */}
              {gradesTabSubjects.filter(sub => !isLiberalArtSubject(sub.name)).length === 0 ? (
                <div className="py-14 text-center">
                  <FileText className="w-10 h-10 text-slate-200 dark:text-slate-800 mx-auto mb-3" />
                  <p className={`font-semibold text-xs leading-relaxed ${isDarkMode ? 'text-slate-500' : 'text-slate-400'}`}>
                    현재 석차등급을 적용 가능한 일반 교과목이 없습니다. <br/>
                    {grade !== '1' && (
                      <span>
                        <strong className="text-blue-500 cursor-pointer" onClick={() => setActiveTab('record')}>기록장 탭</strong>에서 일반 선택과목을 선택 추가해 주세요.
                      </span>
                    )}
                  </p>
                </div>
              ) : (
                <div className="overflow-x-auto">
                  <table className="w-full text-left border-collapse">
                    <thead>
                      <tr className={`border-b text-xs font-black text-slate-400 ${isDarkMode ? 'border-slate-800' : 'border-slate-100'}`}>
                        <th className="py-3 px-2">교과목명</th>
                        <th className="py-3 px-2 w-32">이수 단위 수</th>
                        <th className="py-3 px-2 w-44">현 석차등급 (9등급제)</th>
                        <th className="py-3 px-2 w-44">고교학점제 등급 (5등급제)</th>
                      </tr>
                    </thead>
                    <tbody>
                      {gradesTabSubjects
                        .filter(sub => !isLiberalArtSubject(sub.name))
                        .map((sub) => {
                          const subGrade = grades[sub.id] || { units: '', grade9: '', grade5: '' };
                          return (
                            <tr key={sub.id} className={`border-b text-sm transition-colors ${
                              isDarkMode ? 'border-slate-850 hover:bg-slate-850/40' : 'border-slate-50 hover:bg-slate-50/50'
                            }`}>
                              {/* 과목명 */}
                              <td className="py-4 px-2 font-bold text-slate-800 dark:text-slate-200">
                                {sub.name}
                              </td>

                              {/* 단위 수 입력 */}
                              <td className="py-4 px-2">
                                <input 
                                  type="number" 
                                  min="1" 
                                  max="10"
                                  value={subGrade.units || ''} 
                                  onChange={(e) => updateGradeData(sub.id, 'units', e.target.value)}
                                  placeholder="단위"
                                  className={`w-20 px-2.5 py-1.5 text-xs font-bold rounded-lg border focus:ring-2 focus:ring-blue-500 outline-none transition-all ${
                                    isDarkMode 
                                      ? 'bg-slate-800 border-slate-700 text-white' 
                                      : 'bg-slate-50 border-slate-200 text-slate-800'
                                  }`}
                                />
                              </td>

                              {/* 9등급제 입력 */}
                              <td className="py-4 px-2">
                                <select 
                                  value={subGrade.grade9 || ''} 
                                  onChange={(e) => updateGradeData(sub.id, 'grade9', e.target.value)}
                                  className={`px-2.5 py-1.5 text-xs font-bold rounded-lg border focus:ring-2 focus:ring-blue-500 outline-none transition-all ${
                                    isDarkMode 
                                      ? 'bg-slate-800 border-slate-700 text-white' 
                                      : 'bg-slate-50 border-slate-200 text-slate-800'
                                  }`}
                                >
                                  <option value="">등급 선택</option>
                                  {[1,2,3,4,5,6,7,8,9].map(num => (
                                    <option key={num} value={num}>{num}등급</option>
                                  ))}
                                </select>
                              </td>

                              {/* 5등급제 입력 */}
                              <td className="py-4 px-2">
                                <select 
                                  value={subGrade.grade5 || ''} 
                                  onChange={(e) => updateGradeData(sub.id, 'grade5', e.target.value)}
                                  className={`px-2.5 py-1.5 text-xs font-bold rounded-lg border focus:ring-2 focus:ring-blue-500 outline-none transition-all ${
                                    isDarkMode 
                                      ? 'bg-slate-800 border-slate-700 text-white' 
                                      : 'bg-slate-50 border-slate-200 text-slate-800'
                                  }`}
                                >
                                  <option value="">등급 선택</option>
                                  {[1,2,3,4,5].map(num => (
                                    <option key={num} value={num}>{num}등급</option>
                                  ))}
                                </select>
                              </td>
                            </tr>
                          );
                        })}
                    </tbody>
                  </table>
                </div>
              )}
            </div>
          </div>
        )}

        {/* ==========================================
            [TAB 3: FEEDBACK - AI 피드백 화면]
           ========================================== */}
        {activeTab === 'feedback' && (
          <div className="space-y-6 animate-in fade-in slide-in-from-bottom-3 duration-300">
            
            {/* 1. 학과 검색 및 지정 영역 */}
            <div className={`p-6 rounded-3xl border ${
              isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100 shadow-sm'
            }`}>
              <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-4">
                <div>
                  <h3 className="text-base font-extrabold flex items-center gap-2">
                    <GraduationCap className="w-5 h-5 text-blue-500" />
                    목표 대입 전공 학과 지정
                  </h3>
                  <p className="text-[11px] text-slate-400 font-semibold mt-1">
                    원하는 학과를 선택해 주시면 생기부 기록들을 목표 학과의 학업 및 전공 역량 지표에 맞춰 필터 분석합니다.
                  </p>
                </div>
                
                {/* 현재 설정된 타겟 학과 라벨 */}
                <div className="inline-flex items-center gap-1.5 bg-blue-500/10 text-blue-600 dark:text-blue-400 rounded-xl border border-blue-500/20 text-xs font-black self-start md:self-center">
                  선택된 학과: {selectedDept}
                </div>
              </div>

              {/* 검색창 인풋 및 동적 드롭다운 */}
              <div className="relative">
                <div className="relative">
                  <Search className="absolute left-3.5 top-3 w-4.5 h-4.5 text-slate-400" />
                  <input
                    type="text"
                    value={deptSearch}
                    onChange={(e) => {
                      setDeptSearch(e.target.value);
                      setIsDeptDropdownOpen(true);
                    }}
                    onFocus={() => setIsDeptDropdownOpen(true)}
                    placeholder="원하는 학과명 검색 (예: 컴퓨터, 미디어, 경영, 간호, 의예...)"
                    className={`w-full pl-10 pr-4 py-2.5 border rounded-xl font-bold text-xs transition-all outline-none focus:ring-2 focus:ring-blue-500 ${
                      isDarkMode 
                        ? 'bg-slate-800 border-slate-700 text-white placeholder:text-slate-500' 
                        : 'bg-slate-50 border-slate-200 text-slate-900 placeholder:text-slate-400'
                    }`}
                  />
                  {deptSearch && (
                    <button 
                      onClick={() => { setDeptSearch(''); setIsDeptDropdownOpen(false); }}
                      className="absolute right-3 top-3 text-slate-400 hover:text-slate-600"
                    >
                      <X className="w-4.5 h-4.5" />
                    </button>
                  )}
                </div>

                {/* 자동 필터 리스트 팝업 */}
                {isDeptDropdownOpen && deptSearch && (
                  <div className={`absolute left-0 right-0 mt-1.5 max-h-60 overflow-y-auto z-30 rounded-xl border shadow-xl transition-all ${
                    isDarkMode ? 'bg-slate-900 border-slate-800 text-slate-100' : 'bg-white border-slate-150 text-slate-900'
                  }`}>
                    {filteredDepartments.length === 0 ? (
                      <div className="p-4 text-center text-xs text-slate-400 font-semibold">
                        해당 키워드가 포함된 대입 학과를 찾을 수 없습니다.
                      </div>
                    ) : (
                      filteredDepartments.map((dept, index) => (
                        <button
                          key={index}
                          onClick={() => handleSelectDepartment(dept)}
                          className={`w-full text-left px-4 py-2.5 text-xs font-bold transition-all flex items-center justify-between ${
                            selectedDept === dept 
                              ? 'bg-blue-600/10 text-blue-600 dark:text-blue-400' 
                              : (isDarkMode ? 'hover:bg-slate-800 text-slate-200' : 'hover:bg-slate-50 text-slate-700')
                          }`}
                        >
                          <span>{dept}</span>
                          {selectedDept === dept && <Check className="w-4 h-4 text-blue-500" />}
                        </button>
                      ))
                    )}
                  </div>
                )}
              </div>
            </div>

            {/* 2. AI 분석 리포트 화면 */}
            <div className={`p-8 rounded-3xl border ${
              isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-100 shadow-sm'
            }`}>
              <div className="flex flex-col md:flex-row md:items-center justify-between gap-6 mb-6">
                <div className="flex items-start md:items-center gap-3.5">
                  <div className="w-10 h-10 bg-gradient-to-tr from-blue-600 to-indigo-600 rounded-xl flex items-center justify-center text-white shadow-md shrink-0">
                    <BrainCircuit className="w-5 h-5 animate-pulse" />
                  </div>
                  <div className="space-y-0.5">
                    <h3 className="text-base font-extrabold leading-tight text-slate-800 dark:text-slate-100">Gemini 2.5 AI 실시간 생기부 피드백</h3>
                    <p className="text-[11px] text-slate-400 font-semibold leading-normal">기록장의 내용과 목표 학과의 계열 적합성을 심도 있게 분석합니다.</p>
                  </div>
                </div>

                {/* AI 작동 실행 트리거 버튼 */}
                <button
                  onClick={handleGenerateAIFFEEDBACK || handleGenerateAIFeedback}
                  disabled={isGeneratingFeedback}
                  className="flex items-center justify-center gap-2 px-6 py-3 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 text-white text-xs font-black rounded-xl shadow-lg shadow-blue-100 dark:shadow-none transition-all active:scale-95 disabled:opacity-50 shrink-0 self-start md:self-auto"
                >
                  {isGeneratingFeedback ? (
                    <>
                      <RefreshCw className="w-4 h-4 animate-spin" />
                      생기부 대입 진단 분석 중...
                    </>
                  ) : (
                    <>
                      <Sparkles className="w-4 h-4" />
                      AI 분석 리포트 생성하기
                    </>
                  )}
                </button>
              </div>

              {/* 생성된 마크다운 결과 렌더링 컨테이너 */}
              {isGeneratingFeedback ? (
                <div className="py-20 text-center flex flex-col items-center justify-center gap-4">
                  <div className="relative flex items-center justify-center">
                    <div className="w-16 h-16 rounded-full border-4 border-blue-500/10 border-t-blue-600 animate-spin"></div>
                    <Sparkles className="w-6 h-6 text-blue-500 absolute animate-pulse" />
                  </div>
                  <div className="space-y-1">
                    <p className="text-xs font-bold text-slate-600 dark:text-slate-350">Gemini 생기부 기입 진단 엔진이 작동 중입니다...</p>
                    <p className="text-[10px] text-slate-400 font-semibold">선택한 [{selectedDept}]의 핵심 대입 인재상과 세택 텍스트를 대조하고 있습니다.</p>
                  </div>
                </div>
              ) : aiFeedbackText ? (
                <div className={`p-6 rounded-2xl border transition-all text-xs md:text-sm font-semibold leading-relaxed whitespace-pre-wrap ${
                  isDarkMode ? 'bg-slate-950/40 border-slate-800 text-slate-300' : 'bg-slate-50/50 border-slate-100 text-slate-700'
                }`}>
                  {aiFeedbackText}
                </div>
              ) : (
                <div className={`py-14 text-center rounded-2xl border border-dashed ${
                  isDarkMode ? 'bg-slate-900/30 border-slate-850' : 'bg-slate-50/30 border-slate-200'
                }`}>
                  <BrainCircuit className="w-10 h-10 text-slate-300 dark:text-slate-800 mx-auto mb-3" />
                  <p className={`font-semibold text-xs leading-relaxed ${isDarkMode ? 'text-slate-500' : 'text-slate-400'}`}>
                    목표 학과를 검색 지정한 뒤, <strong className="text-blue-500 font-bold">"AI 분석 리포트 생성하기"</strong> 버튼을 클릭하면 <br/>
                    본인의 전체 생기부 기록장에 기입된 텍스트와 보완책 분석을 수행해 드립니다.
                  </p>
                </div>
              )}
            </div>
          </div>
        )}

        {/* 용남고등학교 CATCH POINT 푸터 로고 영역 */}
        <footer className="mt-16 pt-8 pb-4 border-t border-slate-250/20 dark:border-slate-800/40 text-center animate-in fade-in duration-300">
          <p className="text-[10px] md:text-[11px] font-black tracking-[0.25em] text-slate-400/80 dark:text-slate-600 uppercase select-none">
            용남고등학교 CATCH POINT
          </p>
        </footer>
      </main>

      {/* 설정 팝업 창 (톱니바퀴 클릭 시 모달 노출) */}
      {isSettingsOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm animate-in fade-in duration-200">
          <div className={`rounded-2xl shadow-2xl border p-6 max-w-md w-full animate-in zoom-in-95 duration-200 ${
            isDarkMode ? 'bg-slate-900 border-slate-800 text-slate-100' : 'bg-white border-slate-100 text-slate-900'
          }`}>
            <div className="flex items-center justify-between pb-4 border-b border-slate-100 dark:border-slate-800">
              <h3 className="text-base font-extrabold flex items-center gap-2">
                <Settings className="w-5 h-5 text-blue-500" />
                생기부 기록 매니저 설정
              </h3>
              <button 
                onClick={() => setIsSettingsOpen(false)}
                className={`p-1.5 rounded-lg transition-colors ${isDarkMode ? 'hover:bg-slate-850 text-slate-455' : 'hover:bg-slate-100 text-slate-400'}`}
              >
                <X className="w-4 h-4" />
              </button>
            </div>

            {/* 설정 콘텐츠 영역 */}
            <div className="py-6 space-y-6">
              
              {/* 다크 모드 토글 */}
              <div className="flex items-center justify-between">
                <div>
                  <h4 className="text-sm font-bold">화면 테마 설정</h4>
                  <p className={`text-[11px] mt-0.5 ${isDarkMode ? 'text-slate-400' : 'text-slate-400'}`}>다크모드 및 라이트모드 중 마음에 드는 테마를 선택하세요.</p>
                </div>
                <button 
                  onClick={() => setIsDarkMode(!isDarkMode)}
                  className={`relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${
                    isDarkMode ? 'bg-blue-600' : 'bg-slate-200'
                  }`}
                >
                  <span className={`inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${
                    isDarkMode ? 'translate-x-6' : 'translate-x-1'
                  } flex items-center justify-center`}
                >
                  {isDarkMode ? <Moon className="w-2.5 h-2.5 text-blue-600" /> : <Sun className="w-2.5 h-2.5 text-amber-500" />}
                </span>
                </button>
              </div>

              {/* 글자 측정 모드 선택 */}
              <div className="space-y-2.5">
                <div>
                  <h4 className="text-sm font-bold">글자 수 측정 기준 변경</h4>
                  <p className={`text-[11px] mt-0.5 ${isDarkMode ? 'text-slate-400' : 'text-slate-450'}`}>학교생활기록부 나이스 시스템 등록 기준인 바이트 계산 모드를 켜거나 끌 수 있습니다.</p>
                </div>
                <div className={`grid grid-cols-2 gap-2 p-1 rounded-xl ${isDarkMode ? 'bg-slate-800/80' : 'bg-slate-100'}`}>
                  <button 
                    onClick={() => setByteCalcMode('char')}
                    className={`py-2 text-xs font-bold rounded-lg transition-all ${
                      byteCalcMode === 'char' 
                        ? (isDarkMode ? 'bg-slate-700 text-white shadow-sm' : 'bg-white text-blue-600 shadow-sm') 
                        : 'text-slate-450 hover:text-slate-350'
                    }`}
                  >
                    일반 글자수
                  </button>
                  <button 
                    onClick={() => setByteCalcMode('byte')}
                    className={`py-2 text-xs font-bold rounded-lg transition-all ${
                      byteCalcMode === 'byte' 
                        ? (isDarkMode ? 'bg-slate-700 text-white shadow-sm' : 'bg-white text-blue-600 shadow-sm') 
                        : 'text-slate-400 hover:text-slate-350'
                    }`}
                  >
                    나이스 바이트(Byte)
                  </button>
                </div>
              </div>
            </div>

            <div className="pt-4 border-t border-slate-100 dark:border-slate-800 flex justify-end">
              <button 
                onClick={() => setIsSettingsOpen(false)}
                className="px-5 py-2.5 bg-blue-600 hover:bg-blue-700 text-white rounded-xl font-bold text-xs shadow-md transition-colors"
              >
                설정 완료
              </button>
            </div>
          </div>
        </div>
      )}

      {/* 커스텀 모달 다이얼로그 */}
      {modal.show && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/40 backdrop-blur-sm animate-in fade-in duration-200">
          <div className={`rounded-2xl shadow-xl border p-6 max-w-sm w-full animate-in zoom-in-95 duration-200 ${
            isDarkMode ? 'bg-slate-900 border-slate-800 text-slate-100' : 'bg-white border-slate-100 text-slate-900'
          }`}>
            <h4 className="text-sm font-extrabold flex items-center gap-1.5">
              <AlertCircle className="w-4 h-4 text-red-500" />
              {modal.title}
            </h4>
            <p className={`text-xs mt-2.5 font-semibold leading-relaxed ${isDarkMode ? 'text-slate-450' : 'text-slate-500'}`}>
              {modal.message}
            </p>
            <div className="flex gap-2.5 mt-5">
              <button 
                onClick={modal.onConfirm} 
                className="flex-1 py-2 bg-red-500 hover:bg-red-600 text-white rounded-lg font-bold text-xs transition-colors"
              >
                삭제하기
              </button>
              <button 
                onClick={() => setModal({ show: false, title: '', message: '', onConfirm: null })} 
                className={`flex-1 py-2 rounded-lg font-bold text-xs transition-all ${
                  isDarkMode ? 'bg-slate-850 hover:bg-slate-750 text-slate-455' : 'bg-slate-100 hover:bg-slate-200 text-slate-500'
                }`}
              >
                취소
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

```
