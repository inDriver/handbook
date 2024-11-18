# GoLang Code Review Guide

Code reviews are a crucial part of the software development process, helping to maintain code quality, share knowledge among team members, and improve the overall health of the codebase. Here is a detailed guide for performing code reviews in GoLang.

The primary purpose of code review is to ensure that the overall health of the company’s codebase improves over time. To achieve this, a series of trade-offs must be balanced.

First, developers need to make progress on their tasks. If improvements are never submitted to the codebase, then it cannot improve. Additionally, if a reviewer makes it overly difficult for changes to be approved, developers may become discouraged from making future improvements.

On the other hand, it is the reviewer’s responsibility to ensure that each change list is of a quality that maintains or enhances the overall health of the codebase over time.

A key point to remember is that “perfect” code does not exist—only better code. A change list that, overall, improves the maintainability, readability, and understandability of the system should not be delayed for extended periods simply because it is not “perfect”.


## General

1. Reviewers must understand the context and goal of the change.
2. The code review must start from the perspective of accepting the code, rather than looking for reasons to reject it.
3. Reviewers must provide constructive feedback and be respectful of the author's work.
4. Authors should include enough information in the PR description to make the reviewer's job easier.
5. It’s recommended to split technical tasks, product tasks, and refactoring into separate tasks and PRs to simplify the process.
6. It’s recommended to split large PRs into smaller ones (PR limit is 400 lines) for better comprehension. Note: code-generated files are not included in this limit.
7. Recommended to request changes only for critical issues, such as:
    * violations of the service's overall architecture or established approaches for solving similar issues;
    * evident and significant performance issues;
    * logic errors.


## Code Style and Quality

1. The code must adhere to the established GoLang style guide and project-specific conventions.
2. The code should be clear and easy to understand, avoiding clever tricks and complex structures.
3. The code should be DRY (Don't Repeat Yourself); code duplication should be avoided.
4. The code should follow [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) principles.
5. The code should follow the YAGNI (You Aren't Gonna Need It) and KISS (Keep It Simple, Stupid) principles.
6. The code should not contain any debugging or logging statements left over from development.


## Error Handling

1. All errors must be properly [handled](./programming-practices.md#error-handling). Ignoring returned errors without explanation should be avoided.
2. Use of `panic` should be justified, as Go favors explicit error handling.


## Testing

1. The code must include unit tests that cover both positive and negative cases.
2. Tests must pass before the code can be merged.
3. Mocks should be used for testing functions that depend on external services.
4. Bug fixes should be covered with tests.


## Performance

1. Unnecessary allocations and computations should be avoided.
2. During the review process, if the code performs concurrent operations, reviewers should pay particular attention to these, verifying correct handling of synchronization, data races, and deadlocks.


## Security

1. The code must not contain any potential security issues, such as hardcoded credentials, sensitive data in logs, or risks of SQL injection.
2. Input validation and error messages must be implemented correctly to prevent exposure of internal implementation details.
3. All input must be validated.


## Documentation

1. Publicly exposed APIs, functions, methods, and types should have GoDoc comments that explain their purpose and usage.
2. Complex parts of the code should include comments explaining their functionality.


## Leaving Code Review Comments

Providing feedback in a code review is a skill that develops with practice. Your comments should be helpful, clear, and respectful. Here’s a guide on how to leave constructive code review comments.


### Ask Questions

Rather than simply stating that something is wrong, try asking a question. This approach encourages the author to think about their code from a different perspective and can often lead to a better solution.

```markdown
- Could this function be refactored to avoid repeating this code block?
- I'm not sure, I understand the logic here. Could you explain why you chose this approach?
```


### Be Specific and Provide Examples

Vague feedback can be challenging to act upon. Be specific about what you’re referring to, and, if possible, provide examples or suggestions for how the code could be improved.

```markdown
* this function is doing too much and might be a good candidate for refactoring. Perhaps we could extract this logic into a separate function?
* I see we're duplicating this logic in several places. Could we create a utility function to handle this?
```

### Use a Nitpick

If you’d like to suggest an improvement on a style point not covered in the style guide, prefix your comment with “Nit:” to indicate that it’s a minor suggestion. This lets the developer know it’s a nitpick you think would enhance the code but isn’t mandatory.


### Use a Neutral Tone

It's important to remember that you're reviewing the code, not the author. Avoid using personal pronouns like "you" and "your". Instead, refer to the code itself.

```markdown
* this function could benefit from additional comments.
* this code block seems to handle several different concerns. Could we split it into separate functions for clarity?
```


### Be Respectful and Constructive

Finally, remember that the goal of a code review is to improve the code, not to criticize the author. Be respectful and always provide feedback in a constructive manner.

```markdown
* I think there might be a more efficient way to handle this. Have you considered using a map instead of a slice?
* this function is a bit long and complex. It could be simplified and made easier to read by splitting it into smaller functions.
```

Code reviews are a learning experience for everyone involved. Approach them with an open mind, a positive attitude, and a willingness to learn while helping others learn as well.
