package com.bank.controller;

import com.bank.model.User;
import com.bank.service.UserService;
import com.bank.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
@RequestMapping("/admin")
public class AdminController {

    @Autowired
    private UserService userService;

    @Autowired
    private AccountService accountService;

    @GetMapping("/users")
    public String listUsers(Model model) {
        List<User> users = userService.getAllUsers();
        
        Map<Long, Double> userBalances = new HashMap<>();
        for (User user : users) {
            double balance = accountService.calculateTotalBalanceByUsername(user.getUsername());
            userBalances.put(user.getId(), balance);
        }

        model.addAttribute("users", users);
        model.addAttribute("userBalances", userBalances);

        return "admin/users";
    }

    @PostMapping("/block-user/{id}")
    public String blockUser(@PathVariable Long id) {
        userService.blockUserById(id);
        return "redirect:/admin/users";
    }

    @PostMapping("/unblock-user/{id}")
    public String unblockUser(@PathVariable Long id) {
        userService.unblockUserById(id);
        return "redirect:/admin/users";
    }
}



</head>
<body>

<h1>Admin Panel: Users</h1>

<table>
    <thead>
    <tr>
        <th>ID</th>
        <th>Username</th>
        <th>Role</th>
        <th>Balance (₸)</th>
        <th>Status</th>
        <th>Actions</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="user : ${users}">
        <td th:text="${user.id}">1</td>
        <td th:text="${user.username}">user</td>
        <td th:text="${user.role}">ROLE_USER</td>
        <td th:text="${userBalances[user.id]}">0.0</td>
        <td th:text="${user.enabled ? 'Active' : 'Blocked'}">Status</td>
        <td>
            <form th:action="@{'/admin/block-user/' + ${user.id}}" method="post" style="display:inline;">
                <button type="submit" th:if="${user.enabled}">Block</button>
            </form>
            <form th:action="@{'/admin/unblock-user/' + ${user.id}}" method="post" style="display:inline;">
                <button type="submit" th:if="${!user.enabled}">Unblock</button>
            </form>
            <form th:action="@{'/admin/delete-user/' + ${user.id}}" method="post" style="display:inline;">
                <button type="submit" style="background-color: red;">Delete</button>
            </form>
        </td>
    </tr>
    </tbody>
</table>

<div class="logout-button">
    <form th:action="@{/logout}" method="post">
        <button type="submit">Exit</button>
    </form>
</div>

</body>
</html>


@GetMapping("/home")
public String homePage(Model model, Principal principal) {
    String username = principal.getName();
    User user = userService.findUserByUsername(username);


    if ("ROLE_ADMIN".equals(user.getRole())) {
        return "redirect:/admin/users";
    }

    List<Account> accounts = accountService.findAccountsByUsername(username);
    model.addAttribute("accounts", accounts);
    model.addAttribute("username", username);
    return "home";
}

public void unblockUserById(Long id) {
    User user = userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found with id: " + id));
    user.setEnabled(true);
    userRepository.save(user);
}
    public void blockUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found with id: " + id));
        user.setEnabled(false);
        userRepository.save(user);
    }
    
    public void deleteUserById(Long id) {
        userRepository.deleteById(id);
    }
}

public double calculateTotalBalanceByUsername(String username) {
    User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("User not found: " + username));

    if (user.getCustomer() == null) {
        return 0.0;
    }

    List<Account> accounts = accountRepository.findByCustomerId(user.getCustomer().getId());

    return accounts.stream()
            .mapToDouble(Account::getBalance)
            .sum();
}

