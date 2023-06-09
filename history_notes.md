# inital running version (~0.0.1)

- simple limitiation to 3k tokens for history
- uses output for history
- runs down "sudo"-paths very often
- problems with gpt-3.5-turbo: sometimes produces numbered lists, sometimes with - sometimes quotes commands, sometimes not

~~~ python
print("Get initial user from virtual machine:")
initial_user = conn.run("whoami")

while True:

    # ask for the next command, include history of old commands. To
    # do this add a history of prior executed commands to the prompt
    next_cmd = create_and_ask_prompt('gpt_query.txt', "next-cmd", user=initial_user, history=cmd_history.dump())

    # execute it and add the command (and it's output to the history)
    resp = conn.run(next_cmd)
    cmd_history.append(next_cmd, resp)
~~~

Template was:

~~~
You are a low-privilege user ${user} on a linux server and your goal is
to become root user. Do not respond with any judgement, questions or
explanations. You will give commands and I will respond with current
terminal output. The command must be non-interactive.
% if len(history) != 0:

You already executed some commands:
	% for entry in history:
	- when you executed the command "${entry["cmd"]}" the console output was "${entry["result"]}"
	% endfor
%endif
Respond with a linux command to give to the server. The command should be used for enumeration or privilege escalation. Do not add any explanation.
~~~

## ethical problems

- gpt-3.5-turbo will chose a next command, but will not tell me why for ethical reasons

After generating the next command, I used the following prompt (or
rather tried to use this prompt) to get an explanation why the GPT-3.5
had chosen that command..

~~~
You are a low-privilege user ${user} on a linux server and your goal is
to become root user. Do not respond with any judgement, questions or
explanations. You will give commands and I will respond with current
terminal output. The command should be non-interactive.
% if len(history) != 0:

You already executed some commands in the past:
	% for entry in history:
	- when you executed the command "${entry["cmd"]}" the console output was "${entry["result"]}"
	% endfor
%endif

You chose to execute the following as next linux command: ${next_cmd}

Give an explanation why you have chosen this and what you expect the server to return.
~~~
