# OS Command Injection Lab Summary

## Comparison

| Area | Simple lab | Blind output-redirection lab |
|---|---|---|
| Feature | product stock checker | feedback submission |
| Endpoint | `POST /product/stock` | `POST /feedback/submit` |
| Confirmed parameter | `storeId` | `email` |
| Injection type | direct/verbose | blind |
| First useful evidence | `whoami` output in response | proportional timing delay |
| Final output channel | same HTTP response | file plus separate `/image` request |
| Main side effect | stdout returned directly | stdout redirected to writable file |
| Main difficulty | identifying direct output | separating errors from evidence and finding a side channel |

## Shared mental model

```text
controlled input
-> backend command construction
-> shell interpretation
-> changed OS execution
-> direct output or side effect
```

## Key evidence lessons

### Simple lab

The response contained a normal stock value and the username from `whoami`. The direct output was enough to prove execution.

### Blind lab

A generic `500` response was not enough. Strong evidence came from:

1. milliseconds for normal requests,
2. approximately 4.1 seconds for a five-count ping,
3. approximately 9.3 seconds for a ten-count ping,
4. a file containing the output of `whoami`,
5. successful retrieval of that file through `/image`.

## Main corrections

- Choose parameters through data-flow reasoning, not only validation assumptions.
- Keep facts separate from guesses about the original backend command.
- A response error does not prove command execution.
- Timing should be repeatable and proportional.
- A closing separator may be necessary to isolate an injected command.
- `>` redirects stdout to a file.
- The retrieval endpoint may be a side channel rather than the vulnerable sink.
- Sanitisation is not the primary remediation strategy.

## Developer takeaways

- Avoid operating-system commands when application libraries or APIs are available.
- Avoid shells when process execution is unavoidable.
- Pass a fixed executable and separate validated arguments.
- Consider argument injection even without shell interpretation.
- Apply least privilege, filesystem restrictions, timeouts, output limits, logging, and monitoring.
- Add regression tests that check behaviour and implementation, not only input validation.

## Interview-style answers

### What is OS Command Injection?

It is a vulnerability where untrusted input reaches command or process execution and changes what the operating system runs.

### What is the difference between direct and blind command injection?

Direct injection returns command output in the vulnerable response. Blind injection executes without returning stdout, so timing, files, or outbound interactions are used as evidence.

### What was the root cause in the feedback lab?

The backend allowed a user-controlled email value to influence a shell-interpreted command string.

### Why was the `500` response not enough?

It showed that processing failed, but did not prove that an additional command ran. Repeatable timing and file output proved execution.

### What is the primary fix?

Remove the shell command where possible. Otherwise use a fixed executable and separate validated arguments with the shell disabled, then add allowlists and defence-in-depth controls.
