# quiz-application-using-java

import java.io.*;
import java.util.*;

/**
 * Model class representing a quiz question.
 */
class Question {
    private int id;
    private String questionText;
    private String answer;

    public Question(int id, String questionText, String answer) {
        this.id = id;
        this.questionText = questionText;
        this.answer = answer;
    }

    public int getId() { return id; }
    public String getQuestionText() { return questionText; }
    public String getAnswer() { return answer; }

    public void setQuestionText(String questionText) { this.questionText = questionText; }
    public void setAnswer(String answer) { this.answer = answer; }

    @Override
    public String toString() {
        return id + "|" + questionText + "|" + answer;
    }
}

/**
 * Data Access Object (DAO) class for managing Question persistence.
 */
class QuestionDAO {
    private final File file = new File("data/questions.txt");

    public QuestionDAO() {
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();  // Ensures data directory exists
        }
    }

    public void addQuestion(Question q) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file, true))) {
            writer.write(q.toString());
            writer.newLine();
        }
    }

    public List<Question> getAllQuestions() throws IOException {
        List<Question> list = new ArrayList<>();
        if (!file.exists()) return list;

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split("\\|");
                if (parts.length == 3) {
                    try {
                        int id = Integer.parseInt(parts[0]);
                        list.add(new Question(id, parts[1], parts[2]));
                    } catch (NumberFormatException e) {
                        System.out.println("Skipping invalid question record.");
                    }
                }
            }
        }
        return list;
    }

    public boolean updateQuestion(Question updatedQ) throws IOException {
        List<Question> list = getAllQuestions();
        boolean updated = false;

        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
            for (Question q : list) {
                if (q.getId() == updatedQ.getId()) {
                    writer.write(updatedQ.toString());
                    updated = true;
                } else {
                    writer.write(q.toString());
                }
                writer.newLine();
            }
        }
        return updated;
    }

    public boolean deleteQuestion(int id) throws IOException {
        List<Question> list = getAllQuestions();
        boolean deleted = false;

        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
            for (Question q : list) {
                if (q.getId() == id) {
                    deleted = true;
                    continue;
                }
                writer.write(q.toString());
                writer.newLine();
            }
        }
        return deleted;
    }
}

/**
 * Main Application Class - Quiz Console Application
 */
public class QuizApp {
    private static final Scanner scanner = new Scanner(System.in);
    private static final QuestionDAO dao = new QuestionDAO();

    public static void main(String[] args) {
        System.out.println("====== Welcome to the Quiz App ======");
        boolean running = true;

        while (running) {
            printMenu();
            int choice = getIntInput("Enter your choice: ", 1, 6);
            switch (choice) {
                case 1 -> addQuestion();
                case 2 -> listQuestions();
                case 3 -> updateQuestion();
                case 4 -> deleteQuestion();
                case 5 -> startQuiz();
                case 6 -> {
                    System.out.println("Exiting... Goodbye!");
                    running = false;
                }
                default -> System.out.println("Invalid choice. Try again.");
            }
        }
    }

    private static void printMenu() {
        System.out.println("\n===== Menu =====");
        System.out.println("1. Add Question");
        System.out.println("2. List Questions");
        System.out.println("3. Update Question");
        System.out.println("4. Delete Question");
        System.out.println("5. Take Quiz");
        System.out.println("6. Exit");
    }

    private static void addQuestion() {
        System.out.println("\n-- Add New Question --");
        int id = getIntInput("Enter question ID (integer): ", 1, Integer.MAX_VALUE);
        String questionText = getValidatedString("Enter question text: ");
        String answer = getValidatedString("Enter answer: ");
        Question q = new Question(id, questionText, answer);

        try {
            dao.addQuestion(q);
            System.out.println("✅ Question added successfully!");
        } catch (IOException e) {
            System.out.println("❌ Error saving question: " + e.getMessage());
        }
    }

    private static void listQuestions() {
        System.out.println("\n-- List of Questions --");
        try {
            List<Question> questions = dao.getAllQuestions();
            if (questions.isEmpty()) {
                System.out.println("No questions found.");
                return;
            }
            for (Question q : questions) {
                System.out.printf("ID: %d | Question: %s | Answer: %s%n",
                        q.getId(), q.getQuestionText(), q.getAnswer());
            }
        } catch (IOException e) {
            System.out.println("❌ Error reading questions: " + e.getMessage());
        }
    }

    private static void updateQuestion() {
        System.out.println("\n-- Update Question --");
        int id = getIntInput("Enter ID of question to update: ", 1, Integer.MAX_VALUE);
        String questionText = getValidatedString("Enter new question text: ");
        String answer = getValidatedString("Enter new answer: ");
        Question updated = new Question(id, questionText, answer);

        try {
            boolean success = dao.updateQuestion(updated);
            System.out.println(success ? "✅ Question updated!" : "❌ Question ID not found.");
        } catch (IOException e) {
            System.out.println("❌ Error updating question: " + e.getMessage());
        }
    }

    private static void deleteQuestion() {
        System.out.println("\n-- Delete Question --");
        int id = getIntInput("Enter ID to delete: ", 1, Integer.MAX_VALUE);
        try {
            boolean success = dao.deleteQuestion(id);
            System.out.println(success ? "✅ Question deleted!" : "❌ ID not found.");
        } catch (IOException e) {
            System.out.println("❌ Error deleting question: " + e.getMessage());
        }
    }

    private static void startQuiz() {
        System.out.println("\n-- Starting Quiz --");
        try {
            List<Question> questions = dao.getAllQuestions();
            if (questions.isEmpty()) {
                System.out.println("No questions available.");
                return;
            }

            Collections.shuffle(questions); // Optional enhancement
            int score = 0;
            for (Question q : questions) {
                System.out.println("\nQ: " + q.getQuestionText());
                String userAnswer = getValidatedString("Your answer: ");
                if (userAnswer.equalsIgnoreCase(q.getAnswer())) {
                    System.out.println("✅ Correct!");
                    score++;
                } else {
                    System.out.println("❌ Incorrect. Correct answer: " + q.getAnswer());
                }
            }
            System.out.printf("\n🎯 Quiz complete! Your score: %d/%d%n", score, questions.size());
        } catch (IOException e) {
            System.out.println("❌ Quiz error: " + e.getMessage());
        }
    }

    // --- Input Validation Methods ---

    private static int getIntInput(String prompt, int min, int max) {
        while (true) {
            System.out.print(prompt);
            try {
                int val = Integer.parseInt(scanner.nextLine().trim());
                if (val < min || val > max) {
                    System.out.println("Please enter a valid number between " + min + " and " + max + ".");
                    continue;
                }
                return val;
            } catch (NumberFormatException e) {
                System.out.println("Invalid input! Enter a valid number.");
            }
        }
    }

    private static String getValidatedString(String prompt) {
        while (true) {
            System.out.print(prompt);
            String input = scanner.nextLine().trim();
            if (input.isEmpty()) {
                System.out.println("Input cannot be empty.");
            } else {
                return input;
            }
        }
    }
}
