// UserApp.java
// Java Swing application for user login, registration, and information submission
// Includes admin interface to manage users

import java.awt.*;
import java.awt.event.ActionListener;
import java.io.*;
import javax.swing.*;
import javax.swing.border.Border;

public class UserApp extends JFrame {
    // Layout and main panel setup
    CardLayout card = new CardLayout();
    JPanel mainpanelSection = new JPanel(card);

    // Login fields
    JTextField usernameSection = new JTextField(15);
    JPasswordField loginPassword = new JPasswordField(15);
    JLabel loginStatus = new JLabel();

    // Registration fields
    JTextField registerUsername = new JTextField(15);
    JPasswordField registerPassword = new JPasswordField(15);
    JPasswordField registerConfirmPassword = new JPasswordField(15);
    JTextArea registerstatusSection = new JTextArea();
    JLabel welcomeLabel = new JLabel("Welcome!");

    // User information fields
    JTextField studentnoSection = new JTextField(15);
    JRadioButton maleRadio = new JRadioButton("Male");
    JRadioButton femaleRadio = new JRadioButton("Female");
    ButtonGroup genderGroup = new ButtonGroup();
    JTextField nameField = new JTextField(15);
    JComboBox<String> departmentSection = new JComboBox<>(new String[]{"Computer Engineering", "Software Engineering", "Mathematics", "Physics", "Engineering"});
    JLabel infostatusSection = new JLabel();

    // Stores the currently logged-in user
    String currentUser = "";

    public UserApp() {
        setTitle("User Login Application");
        setSize(400, 350);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Login panel UI
        JPanel loginPanel = new JPanel(new GridLayout(5, 1));
        loginPanel.add(new JLabel("Username:"));
        loginPanel.add(usernameSection);
        loginPanel.add(new JLabel("Password:"));
        loginPanel.add(loginPassword);
        JButton loginBtn = new JButton("Login");
        JButton toRegisterBtn = new JButton("Register");
        loginPanel.add(loginBtn);
        loginPanel.add(toRegisterBtn);
        loginPanel.add(loginStatus);

        // Registration panel UI
        JPanel registerPanel = new JPanel(new GridLayout(5, 1));
        registerPanel.add(new JLabel("New Username:"));
        registerPanel.add(registerUsername);
        registerPanel.add(new JLabel("New Password"));
        registerPanel.add(registerPassword);
        registerPanel.add(new JLabel("Confirm Password"));
        registerPanel.add(registerConfirmPassword);
        JButton registerBtn = new JButton("Register");
        JButton backBtn = new JButton("Back");
        registerstatusSection.setLineWrap(true);
        registerstatusSection.setWrapStyleWord(true);
        registerPanel.add(registerBtn);
        registerPanel.add(backBtn);
        registerPanel.add(registerstatusSection);

        // Admin panel for viewing/deleting users
        JPanel adminPanel = new JPanel(new BorderLayout());
        DefaultListModel<String> userListModel = new DefaultListModel<>();
        JList<String> userList = new JList<>(userListModel);
        JButton deleteUserButton = new JButton("Delete Selected User");
        JButton adminBackButton = new JButton("Back to Login");
        JPanel adminButtonsPanel = new JPanel(new FlowLayout());
        adminButtonsPanel.add(deleteUserButton);
        adminButtonsPanel.add(adminBackButton);
        adminPanel.add(new JLabel("Admin Panel - User Management", SwingConstants.CENTER), BorderLayout.NORTH);
        adminPanel.add(new JScrollPane(userList), BorderLayout.CENTER);
        adminPanel.add(adminButtonsPanel, BorderLayout.SOUTH);

        // Action to go back to login from admin panel
        adminBackButton.addActionListener(e -> {
            usernameSection.setText("");
            loginPassword.setText("");
            loginStatus.setText("");
            card.show(mainpanelSection, "login");
        });

        // Welcome panel after info submission
        JPanel welcomePanel = new JPanel(new BorderLayout());
        JButton welcomeBackButton = new JButton("Back to Info Page");
        welcomePanel.add(welcomeLabel);
        welcomePanel.add(welcomeBackButton, BorderLayout.PAGE_END);

        // Return to info page from welcome panel
        welcomeBackButton.addActionListener(e -> {
            card.show(mainpanelSection, "info");
        });

        // User info input panel
        JPanel infoPanel = new JPanel(new GridLayout(10, 1));
        infoPanel.add(new JLabel("Student Number:"));
        infoPanel.add(studentnoSection);
        infoPanel.add(new JLabel("Gender:"));
        genderGroup.add(maleRadio);
        genderGroup.add(femaleRadio);
        JPanel genderPanel = new JPanel();
        genderPanel.add(maleRadio);
        genderPanel.add(femaleRadio);
        infoPanel.add(genderPanel);
        infoPanel.add(new JLabel("Full Name:"));
        infoPanel.add(nameField);
        infoPanel.add(new JLabel("Department:"));
        infoPanel.add(departmentSection);
        JButton submitInfoBtn = new JButton("Submit");
        infoPanel.add(submitInfoBtn);
        infoPanel.add(infostatusSection);

        // Add panels to card layout
        mainpanelSection.add(loginPanel, "login");
        mainpanelSection.add(registerPanel, "register");
        mainpanelSection.add(welcomePanel, "welcome");
        mainpanelSection.add(adminPanel, "admin");
        mainpanelSection.add(infoPanel, "info");

        add(mainpanelSection);

        // Button actions
        loginBtn.addActionListener(e -> login());
        toRegisterBtn.addActionListener(e -> card.show(mainpanelSection, "register"));
        registerBtn.addActionListener(e -> register());
        backBtn.addActionListener(e -> card.show(mainpanelSection, "login"));
        submitInfoBtn.addActionListener(e -> submitUserInfo());

        setVisible(true);
    }

    // Login logic for users and admin
    void login() {
        String username = usernameSection.getText();
        String password = new String(loginPassword.getPassword());

        if(username.equals("admin") && password.equals("admin")){
            currentUser = "ADMIN";
            loginStatus.setText("");
            loadUserList();
            card.show(mainpanelSection, "admin");
            return;
        }

        if (checkUser(username, password)) {
            currentUser = username;
            card.show(mainpanelSection, "info");
        } else {
            loginStatus.setText("Login failed!");
        }
    }

    // Register a new user with validations
    void register() {
        String username = registerUsername.getText();
        String password = new String(registerPassword.getPassword());
        String confirmPassword = new String(registerConfirmPassword.getPassword());

        if(!password.equals(confirmPassword)){
            registerstatusSection.setText("Passwords do not match!");
            return;
        }
        if (password.length() < 8) {
            registerstatusSection.setText("Password too short.");
            return;
        }
        if (!password.matches(".*[A-Z].*")) {
            registerstatusSection.setText("Password must contain at least one uppercase letter.");
            return;
        }
        if(!password.matches(".*[a-z].*")){
            registerstatusSection.setText("Password must contain at least one lowercase letter.");
            return;
        }
        if(!password.matches(".*[^a-zA-Z0-9].*")){
            registerstatusSection.setText("Password must contain at least one special character.");
            return;
        }
        if(!password.matches(".*[0-9].*")){
            registerstatusSection.setText("Password must contain at least one number.");
            return;
        }
        if (checkUser(username, password)) {
            registerstatusSection.setText("User already exists.");
            return;
        }
        try (FileWriter fw = new FileWriter("users.txt", true)) {
            fw.write(username + ":" + password + "\n");
            registerstatusSection.setText("Registration successful!");
        } catch (IOException e) {
            registerstatusSection.setText("File error.");
        }
    }

    // Load user list into admin panel
    void loadUserList(){
        JPanel adminPanel = (JPanel) mainpanelSection.getComponent(3);
        JScrollPane scrollPane = (JScrollPane) adminPanel.getComponent(1);
        @SuppressWarnings("unchecked")
        JList<String> userList = (JList<String>) scrollPane.getViewport().getView();
        DefaultListModel<String> model = getUserListModel(userList);

        model.clear();

        try (BufferedReader br = new BufferedReader(new FileReader("users.txt"))) {
            String line;
            while((line = br.readLine()) != null) {
                String[] parts = line.split(":");
                if(parts.length >= 2) {
                    model.addElement(parts[0]);
                }
            }
        } catch(IOException e) {
            System.err.println("Error loading users: " + e.getMessage());
        }

        JPanel buttonPanel = (JPanel) adminPanel.getComponent(2); 
        JButton deleteButton = (JButton) buttonPanel.getComponent(0);

        for (ActionListener al : deleteButton.getActionListeners()) {
            deleteButton.removeActionListener(al);
        }

        deleteButton.addActionListener(e -> {
            String selectedUser = userList.getSelectedValue();
            if (selectedUser == null) {
                JOptionPane.showMessageDialog(this, "Please select a user first.");
                return;
            }

            int confirm = JOptionPane.showConfirmDialog(
                this, "Delete user '" + selectedUser + "'?", "Confirm", JOptionPane.YES_NO_OPTION);
            if (confirm != JOptionPane.YES_OPTION) return;

            deleteUserFromFile(selectedUser, "users.txt");
            deleteUserFromFile(selectedUser, "userinfo.txt");
            loadUserList();
        });
    }

    // Retrieve user list model from JList
    private DefaultListModel<String> getUserListModel(JList<String> userList) {
        return (DefaultListModel<String>) userList.getModel();
    }

    // Delete a user from file
    private void deleteUserFromFile(String username, String filename) {
        File file = new File(filename);
        if (!file.exists()) return;

        java.util.List<String> lines = new java.util.ArrayList<>();

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(":");
                if (parts.length >= 1 && !parts[0].equals(username)) {
                    lines.add(line);
                }
            }
        } catch (IOException e) {
            System.err.println("Error reading file: " + e.getMessage());
            return;
        }

        try (PrintWriter writer = new PrintWriter(new FileWriter(file))) {
            for (String line : lines) {
                writer.println(line);
            }
        } catch (IOException e) {
            System.err.println("Error writing file: " + e.getMessage());
        }
    }

    // Check if user credentials are valid
    boolean checkUser(String username, String password) {
        try (BufferedReader br = new BufferedReader(new FileReader("users.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(":");
                if (parts.length == 2 && parts[0].equals(username) && parts[1].equals(password)) {
                    return true;
                }
            }
        } catch (IOException e) {
        }
        return false;
    }

    // Save user's personal information
    void submitUserInfo() {
        String studentNumber = studentnoSection.getText();
        String gender = maleRadio.isSelected() ? "Male" : femaleRadio.isSelected() ? "Female" : "";
        String fullName = nameField.getText();
        String department = (String) departmentSection.getSelectedItem();

        if (studentNumber.isEmpty() || gender.isEmpty() || fullName.isEmpty()) {
            infostatusSection.setText("Please fill all fields.");
            return;
        }

        try (FileWriter fw = new FileWriter("userinfo.txt", true)) {
            fw.write(currentUser + ":" + studentNumber + ":" + gender + ":" + fullName + ":" + department + "\n");
            infostatusSection.setText("Info saved successfully!");
            welcomeLabel.setText("Welcome, " + fullName + "!");
            card.show(mainpanelSection, "welcome");
        } catch (IOException e) {
            infostatusSection.setText("File error.");
        }
    }

    // App entry point
    public static void main(String[] args) {
        new UserApp();
    }
}
