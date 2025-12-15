# Stack Oveflow LLD


## Flow Summary

- **[User](guide://action?prefill=Tell%20me%20more%20about%3A%20User)**: Has `id`, `name`, `UserType`, and `reputation`.
- **[Tag](guide://action?prefill=Tell%20me%20more%20about%3A%20Tag)**: Categorizes questions.
- **[Comment](guide://action?prefill=Tell%20me%20more%20about%3A%20Comment)**: Attached to posts.
- **[Vote](guide://action?prefill=Tell%20me%20more%20about%3A%20Vote)**: Upvote/downvote by users.
- **[Post](guide://action?prefill=Tell%20me%20more%20about%3A%20Post)**: Abstract base for Question/Answer.
- **[Question](guide://action?prefill=Tell%20me%20more%20about%3A%20Question)**: Has title, content, tags, answers.
- **[Answer](guide://action?prefill=Tell%20me%20more%20about%3A%20Answer)**: Linked to a question, can be accepted.
- **[StackOverflow](guide://action?prefill=Tell%20me%20more%20about%3A%20StackOverflow)**: Main system holding users and questions, supports search.



---


## 1. User and Roles

```java
public enum UserType {
    ADMIN, MODERATOR, MEMBER, GUEST
}

public class User {
    private int id;
    private String name;
    private UserType userType;
    private int reputation;

    public User(int id, String name, UserType userType) {
        this.id = id;
        this.name = name;
        this.userType = userType;
        this.reputation = 0;
    }

    public void increaseReputation(int points) {
        this.reputation += points;
    }

    public void decreaseReputation(int points) {
        this.reputation -= points;
    }

    // Permission checks
    public boolean canPostQuestion() {
        return userType != UserType.GUEST;
    }

    public boolean canPostAnswer() {
        return userType != UserType.GUEST;
    }

    public boolean canComment() {
        return userType != UserType.GUEST;
    }

    public boolean canVote() {
        return userType != UserType.GUEST;
    }

    public boolean canAddTag() {
        return userType != UserType.GUEST;
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public UserType getUserType() { return userType; }
    public void setUserType(UserType userType) { this.userType = userType; }
    public int getReputation() { return reputation; }
    public void setReputation(int reputation) { this.reputation = reputation; }
}
```

---

## 2. Tag

```java
public class Tag {
    private int id;
    private String name;
    private String description;

    public Tag(int id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
}
```

---

## 3. Comment

```java
public class Comment {
    private int id;
    private String text;
    private User user;

    public Comment(int id, String text, User user) {
        if (!user.canComment()) {
            throw new IllegalStateException("Guest users cannot add comments");
        }
        this.id = id;
        this.text = text;
        this.user = user;
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getText() { return text; }
    public void setText(String text) { this.text = text; }
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
}
```

---

## 4. Vote

```java
public enum VoteType {
    UPVOTE, DOWNVOTE
}

public class Vote {
    private int id;
    private VoteType voteType;
    private User user;

    public Vote(int id, VoteType voteType, User user) {
        if (!user.canVote()) {
            throw new IllegalStateException("Guest users cannot vote");
        }
        this.id = id;
        this.voteType = voteType;
        this.user = user;

        // Reputation impact
        if (voteType == VoteType.UPVOTE) {
            user.increaseReputation(1);
        } else {
            user.decreaseReputation(1);
        }
    }

    // Getters and Setters
    public int getId() { return id; }
    public VoteType getVoteType() { return voteType; }
    public void setVoteType(VoteType voteType) { this.voteType = voteType; }
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
}
```

---

## 5. Post (Base Class)

```java
import java.util.*;

public abstract class Post {
    protected int id;
    protected String content;
    protected User user;
    protected List<Comment> comments;
    protected List<Vote> votes;

    public Post(int id, String content, User user) {
        this.id = id;
        this.content = content;
        this.user = user;
        this.comments = new ArrayList<>();
        this.votes = new ArrayList<>();
    }

    public void addComment(Comment comment) {
        if (!comment.getUser().canComment()) {
            throw new IllegalStateException("Guest users cannot add comments");
        }
        comments.add(comment);
    }

    public void addVote(Vote vote) {
        if (!vote.getUser().canVote()) {
            throw new IllegalStateException("Guest users cannot vote");
        }
        votes.add(vote);
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
    public List<Comment> getComments() { return comments; }
    public List<Vote> getVotes() { return votes; }
}
```

---

## 6. Question

```java
import java.util.*;

public class Question extends Post {
    private String title;
    private List<Answer> answers;
    private List<Tag> tags;

    public Question(int id, String title, String content, User user) {
        super(id, content, user);
        if (!user.canPostQuestion()) {
            throw new IllegalStateException("Guest users cannot post questions");
        }
        this.title = title;
        this.answers = new ArrayList<>();
        this.tags = new ArrayList<>();
    }

    public void addAnswer(Answer answer) {
        if (!answer.getUser().canPostAnswer()) {
            throw new IllegalStateException("Guest users cannot post answers");
        }
        answers.add(answer);
    }

    public void addTag(Tag tag, User user) {
        if (!user.canAddTag()) {
            throw new IllegalStateException("Guest users cannot add tags");
        }
        tags.add(tag);
    }

    // Getters and Setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public List<Answer> getAnswers() { return answers; }
    public List<Tag> getTags() { return tags; }
}
```

---

## 7. Answer

```java
public class Answer extends Post {
    private Question question;
    private boolean isAccepted;

    public Answer(int id, String content, User user, Question question) {
        super(id, content, user);
        if (!user.canPostAnswer()) {
            throw new IllegalStateException("Guest users cannot post answers");
        }
        this.question = question;
        this.isAccepted = false;
    }

    public void acceptAnswer() {
        this.isAccepted = true;
        this.user.increaseReputation(10); // reward for accepted answer
    }

    // Getters and Setters
    public Question getQuestion() { return question; }
    public void setQuestion(Question question) { this.question = question; }
    public boolean isAccepted() { return isAccepted; }
    public void setAccepted(boolean accepted) { isAccepted = accepted; }
}
```

---

## 8. StackOverflow System (Driver)

```java
import java.util.*;

public class StackOverflow {
    private List<User> users;
    private List<Question> questions;

    public StackOverflow() {
        this.users = new ArrayList<>();
        this.questions = new ArrayList<>();
    }

    public void addUser(User user) {
        users.add(user);
    }

    public void addQuestion(Question question) {
        if (!question.getUser().canPostQuestion()) {
            throw new IllegalStateException("Guest users cannot post questions");
        }
        questions.add(question);
    }

    public List<Question> searchQuestionsByTag(String tagName) {
        List<Question> result = new ArrayList<>();
        for (Question q : questions) {
            for (Tag t : q.getTags()) {
                if (t.getName().equalsIgnoreCase(tagName)) {
                    result.add(q);
                }
            }
        }
        return result;
    }

    // Getters and Setters
    public List<User> getUsers() { return users; }
    public void setUsers(List<User> users) { this.users = users; }
    public List<Question> getQuestions() { return questions; }
    public void setQuestions(List<Question> questions) { this.questions = questions; }
}
```

---





