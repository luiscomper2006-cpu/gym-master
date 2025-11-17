/**
 * Lógica completa do Gym Master para o Construct 3.
 * Versão 100% compatível com TypeScript e sem erros de validação.
 */

interface QuizData {
    question: string;
    options: string[];
    correct: number;
}

interface ExerciseData {
    name: string;
    steps: string[];
    quiz: QuizData;
}

interface RankingEntry {
    name: string;
    score: number;
}

interface AppState {
    currentExerciseName: string;
    currentStepText: string;
    currentQuizOptions: string[];
    currentQuizCorrect: number;
    score: number;
    coins: number;
    ranking: RankingEntry[];
    feedback: string;
    showQuiz: boolean;
    isAnswered: boolean;
    showNextStepButton: boolean;
}

class GymMaster {
    currentExercise: number;
    stepIndex: number;
    score: number;
    coins: number;
    showQuiz: boolean;
    isAnswered: boolean;
    feedback: string;
    ranking: RankingEntry[];
    exercises: ExerciseData[];

    constructor() {
        this.currentExercise = 0;
        this.stepIndex = 0;
        this.score = 0;
        this.coins = 0;
        this.showQuiz = false;
        this.isAnswered = false;
        this.feedback = "Bem-vindo! Comece o treino para receber orientações!";
        this.exercises = this.loadExerciseData();
        this.ranking = this.loadInitialRanking();
    }

    private loadExerciseData(): ExerciseData[] {
        return [
            {
                name: "Agachamento",
                steps: [
                    "Afaste os pés na largura dos ombros.",
                    "Mantenha as costas retas e o olhar à frente.",
                    "Desça como se fosse sentar, sem deixar os joelhos ultrapassarem a ponta dos pés.",
                    "Suba controlando o movimento e contraia os glúteos ao final."
                ],
                quiz: {
                    question: "Qual músculo é mais trabalhado no agachamento?",
                    options: ["Bíceps", "Quadríceps", "Tríceps", "Peitoral"],
                    correct: 1
                }
            },
            {
                name: "Flexão de Braço",
                steps: [
                    "Posicione as mãos na largura dos ombros.",
                    "Mantenha o corpo reto da cabeça aos pés.",
                    "Desça até quase encostar o peito no chão.",
                    "Empurre o corpo de volta para cima, estendendo os braços."
                ],
                quiz: {
                    question: "Qual grupo muscular é o principal na flexão?",
                    options: ["Peitoral", "Tríceps", "Deltóide", "Abdômen"],
                    correct: 0
                }
            }
        ];
    }

    private loadInitialRanking(): RankingEntry[] {
        return [
            { name: "Lucas", score: 20 },
            { name: "Mariana", score: 15 },
            { name: "João", score: 10 }
        ];
    }

    private getCurrentStepText(): string {
        if (this.currentExercise < 0 || this.currentExercise >= this.exercises.length) {
            return "Erro: Exercício não encontrado.";
        }

        const exercise = this.exercises[this.currentExercise];

        if (this.showQuiz) {
            return exercise.quiz.question;
        }

        if (this.stepIndex < exercise.steps.length) {
            return `Passo ${this.stepIndex + 1}: ${exercise.steps[this.stepIndex]}`;
        }

        return "Instruções do exercício concluídas. Próximo passo: Quiz!";
    }

    private updateRanking(): void {
        const playerIndex = this.ranking.findIndex(p => p.name === "Você");

        if (playerIndex === -1) {
            this.ranking.push({ name: "Você", score: this.score });
        } else {
            this.ranking[playerIndex].score = this.score;
        }

        this.ranking.sort((a, b) => b.score - a.score);
    }

    public handleNextStep(): AppState {
        const exercise = this.exercises[this.currentExercise];

        if (this.stepIndex < exercise.steps.length - 1) {
            this.stepIndex++;
            this.feedback = "Boa! Continue seguindo o passo a passo corretamente.";
        } else if (this.stepIndex === exercise.steps.length - 1) {
            this.stepIndex++;
            this.showQuiz = true;
            this.feedback = "Excelente! Vamos ver se você aprendeu mesmo.";
        } else {
            this.feedback = "Responda ao quiz ou avance para o próximo exercício!";
        }

        return this.getAppState();
    }

    public handleAnswer(selectedAnswerIndex: number): AppState {
        if (this.isAnswered || !this.showQuiz) {
            this.feedback = "O quiz já foi respondido ou não está visível!";
            return this.getAppState();
        }

        const exercise = this.exercises[this.currentExercise];
        this.isAnswered = true;

        if (selectedAnswerIndex === exercise.quiz.correct) {
            const earnedCoins = 10;
            this.score += 10;
            this.coins += earnedCoins;
            this.feedback = `Perfeito! Você acertou e ganhou ${earnedCoins} moedas.`;
            this.updateRanking();
        } else {
            const correctOption = exercise.quiz.options[exercise.quiz.correct];
            this.feedback = `Quase! A resposta correta era: "${correctOption}". Revise o exercício.`;
        }

        return this.getAppState();
    }

    public nextExercise(): AppState {
        this.currentExercise = (this.currentExercise + 1) % this.exercises.length;
        this.stepIndex = 0;
        this.showQuiz = false;
        this.isAnswered = false;
        this.feedback = "Iniciando novo exercício! Pronto para o próximo desafio?";
        return this.getAppState();
    }

    public getAppState(): AppState {
        const exercise = this.exercises[this.currentExercise];
        return {
            currentExerciseName: exercise.name,
            currentStepText: this.getCurrentStepText(),
            currentQuizOptions: exercise.quiz.options,
            currentQuizCorrect: exercise.quiz.correct,
            score: this.score,
            coins: this.coins,
            ranking: this.ranking,
            feedback: this.feedback,
            showQuiz: this.showQuiz,
            isAnswered: this.isAnswered,
            showNextStepButton: !this.showQuiz || this.isAnswered
        };
    }
}

// --- Instância global para o Construct 3 ---
const gymMasterApp = new GymMaster();

(window as any).initGymMaster = (): AppState => {
    console.log("Gym Master - Lógica inicializada com sucesso!");
    return gymMasterApp.getAppState();
};

(window as any).getGymMasterState = (): AppState => {
    return gymMasterApp.getAppState();
};

(window as any).advanceStep = (): AppState => {
    return gymMasterApp.handleNextStep();
};

(window as any).answerQuiz = (index: number): AppState => {
    const selectedAnswerIndex = parseInt(index as any, 10);
    if (isNaN(selectedAnswerIndex)) {
        console.error("Erro: Índice de resposta inválido recebido.");
        return gymMasterApp.getAppState();
    }
    return gymMasterApp.handleAnswer(selectedAnswerIndex);
};

(window as any).moveToNextExercise = (): AppState => {
    return gymMasterApp.nextExercise();
};

