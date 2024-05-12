import json
import os

import pandas as pd
import streamlit as st

exams = {}
filename = 'exam.json'
if os.path.exists(filename):
    with open(filename, 'r') as f:
        exams = json.load(f)

if 'code' not in st.session_state:
    st.session_state['code'] = ''
if 'ans' not in st.session_state:
    st.session_state['ans'] = ''


def new():
    if exams != {}:
        st.subheader('Modify exist exam')
        for k, v in exams.items():
            temp_ans = v['ans']
            if k and st.button(f'Exam {k} : {temp_ans}'):
                st.session_state['code'] = k
                st.session_state['ans'] = temp_ans

    st.subheader('Create new exam')
    code = st.text_input('Exam code', value=st.session_state['code'])
    ans = st.text_input('Answer key', value=st.session_state['ans'])
    num_q = st.number_input('Number of questions', step=1, value=len(ans))
    exams[code] = {'numq': num_q, 'ans': ans}
    if st.button('Save'):
        if len(ans) == num_q:
            with open(filename, 'w') as f:
                json.dump(exams, f)
            st.success(f'Exam {code} saved.')
        else:
            st.error('Error: Answer key length does not match number of questions.')


def grade():
    code = st.selectbox('Select exam code', list(exams.keys()))
    if code:
        name = st.text_input('Student name')
        stud = st.text_input(
            f'Student answers as a string, with spaces for unanswered questions'
        )
        if st.button('Save Answer'):
            ans = exams[code]['ans']
            num_q = exams[code]['numq']

            if len(stud) == num_q:
                num_correct = sum([1 for i in range(num_q) if stud[i] == ans[i]])
                grade = num_correct / num_q * 10
                st.success(
                    f'{name} got {num_correct} out of {num_q} correct, grade {grade}'
                )

                csv = 'grade.csv'
                data = {
                    'Name': [name],
                    'Exam Code': [code],
                    'Number Correct': [num_correct],
                    'Total Questions': [num_q],
                    'Grade': [grade],
                }
                if os.path.exists(csv):
                    df = pd.read_csv(csv)
                    df = pd.concat([df, pd.DataFrame(data)])
                    df.sort_values('Name', inplace=True)
                    df.to_csv(csv, index=False)
                else:
                    pd.DataFrame(data).to_csv(csv, index=False)
            else:
                st.warning(
                    f'Invalid number of answers: expected {num_q}, got {len(stud)}'
                )


st.sidebar.title('Multiple Choice Exam Grader')
page = st.sidebar.selectbox('Select an option', ('Add Exam', 'Grade Exam'))

if page == 'Add Exam':
    new()
else:
    grade()
