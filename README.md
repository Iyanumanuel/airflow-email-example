# E-mail notifications with Airflow

E-mails can be sent notifying the status of tasks(such as failure, retry) or by custom e-mailing tasks. Let’s take a look at two simple examples for these two cases. We are using the *airflow.utils.email* and *airflow.operators.email_operator* — which is also based on the former.

# Adding e-mail server configuration
Following the Apache Airflows “universal order of precedence” for setting configurations given here, I added the SMTP configurations the “smoothest” possible way, which is by setting as environment variables. As an example, if you use gmail, the configuration would look something like this:
```
AIRFLOW__SMTP__SMTP_HOST=smtp.gmail.com
AIRFLOW__SMTP__SMTP_PORT=587
AIRFLOW__SMTP__SMTP_USER=your-mail-id@gmail.com
AIRFLOW__SMTP__SMTP_PASSWORD=yourpassword
AIRFLOW__SMTP__SMTP_MAIL_FROM=your-mail-id@gmail.com
```

However, gmail would block sending emails using unknown apps. Therefore its better to use a different host to try this. If you prefer to use gmail, the approach explained [here](https://stackoverflow.com/questions/51829200/how-to-set-up-airflow-send-email), should work, though I haven’t tried it.

You can verify these variables from ‘Admin -> Configuration’ on the UI.

# Case 1: Sending a custom email using e-mail operator
```python
from airflow import DAG
from airflow.operators.email_operator import EmailOperator

t1 = EmailOperator(
    task_id="send_mail", 
    to='receiver@mail.com',
    subject='Test mail',
    html_content='<p> You have got mail! <p>',
    dag=dag)
```

# Case 2: Sending e-mail notification on task failure
Here, we’ve set the ‘email_on_failure’ to True, and ‘email’ to recipients address for the operator. For demonstrating, a function which raises an exception is called, which when invoked will result in marking the task (t2) as failed.

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def error_function():
    raise Exception('Something wrong')
   
t2 = PythonOperator(
    task_id='failing_task',
    python_callable=error_function,
    email_on_failure=True,
    emai='receivers@mail.com
    dag=dag,
)
```
