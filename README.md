# internet-programming
http://127.0.0.1:5500/feedback_form.html

feedback_form html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Feedback Form</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f9;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .feedback-form {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 300px;
        }
        .feedback-form h2 {
            margin-bottom: 20px;
            color: #333;
        }
        .feedback-form label {
            display: block;
            margin-bottom: 8px;
            color: #555;
        }
        .feedback-form input, .feedback-form textarea, .feedback-form select {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .feedback-form button {
            width: 100%;
            padding: 10px;
            background-color: #28a745;
            border: none;
            border-radius: 4px;
            color: white;
            font-size: 16px;
        }
        .feedback-form button:hover {
            background-color: #218838;
        }
        .error {
            color: red;
            margin-bottom: 10px;
        }
    </style>
    <script>
        function validateForm() {
            let isValid = true;
            let errorMessages = "";
            const name = document.forms["feedbackForm"]["name"].value;
            const email = document.forms["feedbackForm"]["email"].value;
            const feedback = document.forms["feedbackForm"]["feedback"].value;
            const rating = document.forms["feedbackForm"]["rating"].value;

            if (name === "") {
                errorMessages += "Name must be filled out.\n";
                isValid = false;
            }
            if (email === "") {
                errorMessages += "Email must be filled out.\n";
                isValid = false;
            } else {
                const emailPattern = /^[^@\s]+@[^@\s]+\.[^@\s]+$/;
                if (!emailPattern.test(email)) {
                    errorMessages += "Invalid email format.\n";
                    isValid = false;
                }
            }
            if (feedback === "") {
                errorMessages += "Feedback must be filled out.\n";
                isValid = false;
            }
            if (rating === "" || isNaN(rating) || rating < 1 || rating > 5) {
                errorMessages += "Rating must be a number between 1 and 5.\n";
                isValid = false;
            }

            if (!isValid) {
                alert(errorMessages);
            }

            return isValid;
        }
    </script>
</head>
<body>
    <form class="feedback-form" name="feedbackForm" action="submit_feedback.php" method="POST" onsubmit="return validateForm()">
        <h2>Feedback Form</h2>
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required>
        
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        
        <label for="feedback">Feedback:</label>
        <textarea id="feedback" name="feedback" rows="4" required></textarea>
        
        <label for="rating">Rating (1-5):</label>
        <input type="number" id="rating" name="rating" min="1" max="5" required>
        
        <button type="submit">Submit</button>
    </form>
</body>
</html>

submit feedback_php

<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = htmlspecialchars($_POST['name']);
    $email = htmlspecialchars($_POST['email']);
    $feedback = htmlspecialchars($_POST['feedback']);
    $rating = (int)$_POST['rating'];
    $servername = "localhost"; 
    $username = "root"; 
    $password = ""; 
    $dbname = "campaign_feedback";
    $conn = new mysqli($servername, $username, $password, $dbname);
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    $stmt = $conn->prepare("INSERT INTO feedback (name, email, feedback, rating, submission_date) VALUES (?, ?, ?, ?, NOW())");
    if ($stmt) {
        $stmt->bind_param("sssi", $name, $email, $feedback, $rating);
        if ($stmt->execute()) {
            echo "Feedback submitted successfully!";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
    } else {
        echo "Error preparing statement: " . $conn->error;
    }
    $conn->close();
} else {
    echo "Invalid request method.";
}
?>
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = $_POST['name'];
    $email = $_POST['email'];
    $feedback = $_POST['feedback'];
    $rating = $_POST['rating'];
    $servername = "phpmyadmin"; 
    $username = "submit_feedback"; 
    $password = ""; 
    $dbname = "campaign_feedback";
    $conn = new mysqli($servername, $username, $password, $dbname);
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    $stmt = $conn->prepare("INSERT INTO feedback (name, email, feedback, rating, submission_date) VALUES (?, ?, ?, ?, NOW())");
    $stmt->bind_param("sssi", $name, $email, $feedback, $rating);
    if ($stmt->execute()) {
        echo "Feedback submitted successfully!";
    } else {
        echo "Error: " . $stmt->error;
    }
    $stmt->close();
    $conn->close();
} else {
    echo "Invalid request method.";
}
?>

view_feedback.php

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>View Feedback</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f9;
            margin: 0;
            padding: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th, td {
            padding: 12px;
            border: 1px solid #ddd;
            text-align: left;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .pagination {
            display: flex;
            justify-content: center;
        }
        .pagination a {
            color: black;
            padding: 8px 16px;
            text-decoration: none;
            border: 1px solid #ddd;
            margin: 0 4px;
        }
        .pagination a.active {
            background-color: #4CAF50;
            color: white;
            border: 1px solid #4CAF50;
        }
        .pagination a:hover:not(.active) {
            background-color: #ddd;
        }
    </style>
</head>
<body>
    <h1>Feedback Submissions</h1>
    <?php
    $servername = "phpmyadmin"; 
    $username = "submit_feedback";
    $password = ""; 
    $dbname = "campaign_feedback";
    $conn = new mysqli($servername, $username, $password, $dbname);
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    $results_per_page = 10;
    if (isset($_GET['page']) && is_numeric($_GET['page'])) {
        $current_page = (int)$_GET['page'];
    } else {
        $current_page = 1;
    }
    $start_from = ($current_page - 1) * $results_per_page;
    $sql = "SELECT * FROM feedback LIMIT $start_from, $results_per_page";
    $result = $conn->query($sql);
    if 
        echo "<table>";
        echo "<tr><th>ID</th><th>Name</th><th>Email</th><th>Feedback</th><th>Rating</th><th>Submission Date</th></tr>";
        while($row = $result->fetch_assoc()) {
            echo "<tr>
                    <td>" . $row["id"] . "</td>
                    <td>" . htmlspecialchars($row["name"]) . "</td>
                    <td>" . htmlspecialchars($row["email"]) . "</td>
                    <td>" . htmlspecialchars($row["feedback"]) . "</td>
                    <td>" . htmlspecialchars($row["rating"]) . "</td>
                    <td>" . $row["submission_date"] . "</td>
                  </tr>";
        }
        echo "</table>";
    } else {
        echo "No feedback found.";
    } 
    $sql_total = "SELECT COUNT(id) AS total FROM feedback";
    $result_total = $conn->query($sql_total);
    $row_total = $result_total->fetch_assoc();
    $total_pages = ceil($row_total["total"] / $results_per_page); 
    if ($total_pages > 1) {
        echo '<div class="pagination">';
        for ($i = 1; $i <= $total_pages; $i++) {
            if ($i == $current_page) {
                echo '<a class="active" href="view_feedback.php?page=' . $i . '">' . $i . '</a>';
            } else {
                echo '<a href="view_feedback.php?page=' . $i . '">' . $i . '</a>';
            }
        }
        echo '</div>';
    }
    $conn->close();
    ?>
</body>
</html>


