---
layout: post
title:  "python context manager 实现email发送"
date:   2019-05-05 10:27:02 +0800
comments: true
tags:
- python
- context manager
- email
---

```
#encoding=utf-8
"""
@version: 1
@author: Duke
@file:monitor.py
@time:2019/4/30 10:43 AM
"""

from contextlib import contextmanager
from collections import defaultdict
import luigi
import threading
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import datetime

EMAIL = ""
PASSWORD = ""
SCOPE = list()
WITH_HTML = True

class Monitor:
    recorded_events = defaultdict(list)
    notify_events = None
    core_module = None
    root_task = None
    root_task_parameters = None

    def is_success_only(self):
        success_only = True
        for k, i in self.recorded_events.items():
            if k == 'SUCCESS' and len(i) > 0:
                success_only = success_only and True
            elif len(i) > 0:
                success_only = success_only and False
                break
        return success_only

    def has_missing_tasks(self):
        return True if self.recorded_events['DEPENDENCY_MISSING'] else False

    def has_failed_tasks(self):
        return True if self.recorded_events['FAILURE'] else False

m = Monitor()

//luigi success事件处理函数
def success(task):
    global SCOPE
    task = str(task)
    m.recorded_events['FAILURE'] = \
        [failure for failure in m.recorded_events['FAILURE'] if task not in
         failure['task']]
    for item in SCOPE:
        if item in task:
            m.recorded_events['SUCCESS'].append(task)

//luigi failure事件处理函数
def failure(task, exception):
    task = str(task)
    failure = {'task': task, 'exception': str(exception)}
    m.recorded_events['FAILURE'].append(failure)

event_map = {
    "FAILURE": {"function": failure, "handler": luigi.Event.FAILURE},
    "SUCCESS": {"function": success, "handler": luigi.Event.SUCCESS}
}

def set_handlers(events):
    if not isinstance(events, list):
        raise Exception("events must be a list")

    for event in events:
        if not event in event_map:
            raise Exception("{} is not a valid event.".format(event))
        handler = event_map[event]['handler']
        function = event_map[event]['function']
        // 调用luigi事件处理接口
        luigi.Task.event_handler(handler)(function)


def format_data(data, format):
    result = ""
    if m.recorded_events['FAILURE']:
        result += "Failure Task:\n"
        for item in m.recorded_events['FAILURE']:
            result += item.get("task", "") + "----->" + \
                      item.get("exception", "") + "\n"
        result += "--------------------------------\n"

    if m.recorded_events['SUCCESS']:
        result = result + "Success Task:\n"
        for item in m.recorded_events['SUCCESS']:
            result = result + item + "\n"
        result = result + "--------------------------------\n"
    if data:
        result = result + "Data details:\n"
        result = result + "                   ".join(format) + "\n"
        for item in data:
            value = [str(item.get(key)) for key in format]
            result += "    ".join(value) + "\n"
    print result
    return result


def format_data_with_html(data, format_list):
    // html 渲染email数据， 使用表格等html元素来装饰数据
    FILTER_PROJECT = []

    result = "<html><body>"
    result_suffix = "</body></html>"
    result_head_prefix = "<h3>"
    result_head_suffix = "</h3>"
    result_table_prefix = '<table border="1" cellspacing="0">'
    result_table_suffix = "</table>"
    result_th_prefix = "<th>"
    result_th_suffix = "</th>"
    result_tr_prefix = "<tr>"
    result_tr_suffix = "</tr>"
    result_td_prefix = "<td>"
    result_td_suffix = "</td>"
    result_special_prefix = '<i style="color:red">'
    result_special_suffix = "</i>"
    if m.recorded_events['FAILURE']:
        result += result_head_prefix + "Failure Task:" + result_head_suffix \
                  + result_table_prefix
        for item in m.recorded_events['FAILURE']:
            result += result_tr_prefix + \
                      result_td_prefix + item.get("task", "") + result_td_suffix + \
                      result_td_prefix + item.get("exception", "") + result_td_suffix + \
                      result_tr_suffix
        result += result_table_suffix

    if m.recorded_events['SUCCESS']:
        result = result + result_head_prefix + "Success Task:" + \
                 result_head_suffix + result_table_prefix
        for item in m.recorded_events['SUCCESS']:
            result = result + result_tr_prefix + item + result_tr_suffix
        result = result + result_table_suffix
    if data:
        result = result + result_head_prefix + "Data details:" + \
                 result_head_suffix + result_table_prefix
        for i in format_list:
            if i == "today":
                result = result + result_th_prefix + data[0]["date"].split(" ")[0] + result_th_suffix
                continue
            if i == "yesterday":
                tmp = data[0]["date"].split(" ")[0].split("-")
                if len(tmp) != 3:
                    yesterday_date = ""
                else:
                    yesterday_date = "{date:%Y-%m-%d}".format(
                        date=datetime.datetime(
                            int(tmp[0]), int(tmp[1]), int(tmp[2])) -
                             datetime.timedelta(days=1))
                result = result + result_th_prefix + yesterday_date + result_th_suffix
                continue
            if i == "average":
                result = result + result_th_prefix + "Last week Avg" + result_th_suffix
                continue
            if i != "date":
                result = result + result_th_prefix + i + result_th_suffix

        for item in data:
            if item.get("project_code", None) in FILTER_PROJECT:
                continue
            result += result_tr_prefix
            for key in format_list:
                if key == "today" and item.get(key, None) is not None and \
                                item.get("yesterday", None) is not None and \
                                        item["yesterday"] - item[key] > 0.1:
                    tmp = item.get(key, "")
                    tmp_data = str(float(format(tmp, ".4f")) * 100) + r"%" if \
                        isinstance(tmp, float) else ""
                    result += result_td_prefix + result_special_prefix + \
                              tmp_data + result_special_suffix + \
                              result_td_suffix
                    continue
                if (key == "today" or key == "yesterday" or key == "average")\
                        and item.get(key, None) is not None and item[key] < 0.98:
                    tmp = item.get(key, "")
                    tmp_data = str(float(format(tmp, ".4f"))*100) + r"%" if \
                        isinstance(tmp, float) else ""
                    result += result_td_prefix + result_special_prefix + \
                              tmp_data + result_special_suffix + \
                              result_td_suffix
                    continue
                if key == "date" or key == "project_code":
                    if key != "date":
                        result += result_td_prefix + str(item.get(key, "")) + \
                                  result_td_suffix
                else:
                    tmp = item.get(key, "")
                    tmp_data = str(float(format(tmp, ".4f")) * 100) + r"%" if \
                        isinstance(tmp, float) else ""
                    result += result_td_prefix + tmp_data + result_td_suffix
            result += result_tr_suffix
        result += result_table_suffix
    result += result_suffix
    return result


def send_message(subject, to, data, format):
    if not WITH_HTML:
        data = format_data(data, format)
        message = """From: %s\nTo: %s\nSubject: %s\n\n%s
            """ % (EMAIL, ", ".join(to), subject, data)
    else:
        //使用email库和smtplib库实现email发送 
        msg = MIMEMultipart()
        msg['Subject'] = subject
        msg['From'] = EMAIL
        msg['To'] = ",".join(to)
        html = format_data_with_html(data, format)
        context = MIMEText(html, _subtype='html', _charset='utf-8')  # 解决乱码
        msg.attach(context)
        message = msg.as_string()
    try:
        server = smtplib.SMTP("smtp.gmail.com", 587)
        server.set_debuglevel(1)
        server.ehlo()
        server.starttls()
        server.login(EMAIL, PASSWORD)
        server.sendmail(EMAIL, to, message)
        server.close()
    except Exception, e:
        raise e

// 入口， 使用context manager装饰， 供with语句调用
@contextmanager
def monitor(events, success_report_task, subject, to, data, format):

    with threading.Lock():
        global SCOPE
        SCOPE = success_report_task

    if events:
        // 设置luigi事件监听
        m.notify_events = events
        set_handlers(events)
    yield m
    data = sorted(data, key=lambda tmp: tmp["project_code"])
    send_message(subject,to=to,data=data,
                 format=format)
```
