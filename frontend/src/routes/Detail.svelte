<script>
    import fastapi from "../lib/api";
    import Error from "../components/Error.svelte";
    import { link, push } from 'svelte-spa-router';
    import { is_login, username } from "../lib/store"
    import moment from 'moment/min/moment-with-locales'
    moment.locale('ko')

    export let params = {}
    let question_id = params.question_id
    let question = {answers:[], voter:[]}
    let content = ""
    let error = {detail:[]}

    function get_question() {
        fastapi("get", "/question/" + question_id, {}, (json) => {
            question = json
        })
    }

    get_question()

    function post_answer(event) {
        event.preventDefault()
        let url = "/answer/create/" + question_id
        let params = {
            content: content
        }
        fastapi('post', url, params, 
            (json) => {
                content = ''
                error = {detail:[]}
                get_question()
            },
            (err_json) => {
                error = err_json
            }
        )
    }

    function delete_question(_question_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/question/" + _question_id
            let params = {
                question_id: _question_id
            }
            fastapi('delete', url, params, 
                (json) => {
                    push('/')
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function delete_answer(answer_id) {
        if(window.confirm('정말로 삭제하시겠습니까?')) {
            let url = "/answer/delete/" + answer_id
            let params = {
                answer_id: answer_id
            }
            fastapi('delete', url, params, 
                (json) => {
                    get_question()
                },
                (err_json) => {
                    error = err_json
                }
            )
        }
    }

    function vote_question(_question_id) {
        let url = "/question/"+ _question_id +"/vote"
        fastapi('post', url, 
            (json) => {
                get_question();
            },
            (err_json) => {
                error = err_json;
            }
        );
    }

    function vote_answer(answer_id) {
        let url = "/answer/" + answer_id + "/vote"
        fastapi('post', url, 
            (json) => {
                get_question();
            },
            (err_json) => {
                error = err_json
            }
        )
    }

</script>
<div class="container my-3">
    <!-- 질문 -->
    <h2 class="border-bottom py-2">{question.subject}</h2>
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{question.content}</div>
            <div class="d-flex justify-content-end">
                <div class="badge bg-light text-dark p-2 mt-3">
                    {moment(question.create_date).format("YYYY-MM-DD HH:mm")}
                </div>
                {#if question.modify_date}
                <div class="badge bg-light text-dark p-2 mt-3">
                    {moment(question.modify_date).format("YYYY-MM-DD HH:mm")}
                </div>
                {/if}
            </div>
            <div class="my-3">
                <button class="btn btn-sm btn-outline-secondary"
                    on:click="{vote_question(question.id)}"> 
                    추천
                    <span class="badge rounded-pill bg-success">{ question.voter.length }</span>
                </button>
                {#if question.user && $username === question.user.username }
                <a use:link href="/question-modify/{question.id}" 
                    class="btn btn-sm btn-outline-secondary">수정</a>
                <button class="btn btn-sm btn-outline-danger"
                on:click={() => delete_question(question.id)}>삭제</button>
                {/if}
            </div>
        </div>
    </div>

    <button class="btn btn-secondary" on:click="{() => {
        push('/')
    }}">목록으로</button>


    <!-- 답변 목록 -->
    <h5 class="border-bottom my-3 py-2">{question.answers.length}개의 답변이 있습니다.</h5>
    {#each question.answers as answer}
    <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{answer.content}</div>
            <div class="d-flex justify-content-end">
                <div class="badge bg-light text-dark p-2 mt-3 text-start">
                    <div class="mb-2">작성자: { question.user ? question.user.username : ""}</div>
                    <div class="mb-2">작성일: {moment(answer.create_date).format("YYYY-MM-DD HH:mm")}</div>
                    {#if answer.modify_date}
                    <div class="mb-2">수정일: {moment(answer.modify_date).format("YYYY-MM-DD HH:mm")}</div>
                    {/if}
                </div>
            </div>
            <div class="my-3">
                <button class="btn btn-sm btn-outline-secondary"
                    on:click="{vote_answer(answer.id)}"> 
                    추천
                    <span class="badge rounded-pill bg-success">{ answer.voter.length }</span>
                </button>
                {#if answer.user && $username === answer.user.username }
                <a use:link href="/answer-modify/{answer.id}" 
                    class="btn btn-sm btn-outline-secondary">수정</a>
                <button class="btn btn-sm btn-outline-danger"
                on:click={() => delete_answer(answer.id) }>삭제</button>
                {/if}
            </div>
        </div>
    </div>
    {/each}
    <!-- 답변 등록 -->
    <Error error={error} />
    <form method="post" class="my-3">
        <div class="mb-3">
            <textarea rows="10" bind:value={content}
                disabled={$is_login ? "" : "disabled"}
                class="form-control" />
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary {$is_login ? '' : 'disabled'}" on:click="{post_answer}" />
    </form>
</div>

