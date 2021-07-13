---
title: Deno ve React ile Not Uygulaması
---

![]({{ 'Screen-Recording-_17.05.2020-17-20-09_.gif' | relative_url }})


Son zamanalarda duyduğum bir programlama dili olan Deno ile bir örnek yapmak istedim. Aslında ben Python geliştiricisi olarak, Javascript ve TypeScript dilleri hakkında pek bilgi sahibi değilim; ama bugünlerde boş durmak olmaz diyerek, bu yeni programlama diliyle biraz uğraşmak istedim.


Dataları depolamak için [json-server](https://github.com/typicode/json-server) kullandım.


Projenin yapısı; 
<br>

![]({{ 'Screenshot_1.png' | relative_url }})


<b>config.js</b>
{% highlight javascript %}

const config = {
    HOST : "127.0.0.1",
    PORT: 9191
}
{% endhighlight %}




<b>mod.js</b>

{% highlight javascript %}

import {Application, send, Router} from "oak";
import config  from './config.js'

const app = new Application();

app.use( async(context) => {
    await send(context, context.request.url.pathname,{
        root: `${Deno.cwd()}/react/`,
        index: "app.html"
    })
})

await app.listen({port : config.PORT})

{% endhighlight %}

Yukarıdaki kod ile server'ı ayağa kaldırıyoruz.


<b>react/index.html</b>

{% highlight html %}

<html>
    <head>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossOrigin="anonymous" />
        <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossOrigin="anonymous"></script>
        <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossOrigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossOrigin="anonymous"></script>
    </head>
    <body>
        <div id="app"></div>        

    </body>

    <!-- Load React. -->
    <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
    <script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
    

    <script src="./context.js" type='text/babel'></script>
    <script src="./components//navbar.js" type='text/babel'></script>
    <script src="./components/modal.js" type='text/babel'></script>
    <script src="./components/card.js" type='text/babel'></script>
    <script src="./components/cards.js" type='text/babel'></script>
    <script src="./app.js" type='text/babel'></script>
    

</html>
{% endhighlight %}

<b> react/app.js </b>

{% highlight javascript %}

class App extends React.Component {

    render(){
        return (
            <React.Fragment>
                <Navbar />
                <Modal />
                <NotesCard />
            </React.Fragment>
        )
    }
}

ReactDOM.render(
    <Provider>
        <App />
    </Provider>
    , document.getElementById("app"));

{% endhighlight %}



<b>react/context.js</b>
{% highlight javascript %}



const Context = React.createContext();


const reducer = (state, action) => {
    switch(action.type){
        case "DELETE_NOTE":
            return {
                ...state,
                notes: state.notes.filter(note => note.id !== action.payload)
            }
        case "ADD_NOTE":
            return {
                ...state,
                notes: [...state.notes, action.payload]
            }
            
    }
}


class Provider extends React.Component {
    state = {
        notes: [],
        dispatch : (action)=> {
            this.setState(state=> reducer(state,action));
        }
    }

    componentDidMount(){
        fetch("http://127.0.0.1:9999/notes")
            .then(res => res.json())
            .then(
                (result) => {
                    this.setState({notes: result})
                }
            )
    }

    render(){
        return(
            <Context.Provider value={this.state}>
                {this.props.children}
            </Context.Provider>
        )
    }
}

const Consumer = Context.Consumer;




{% endhighlight %}



<b>react/components/card.js</b>
{% highlight javascript %}

class Note extends React.Component {
    
    deleteNote = (dispatch, e) => {
        const {id} = this.props;
       dispatch({ type: "DELETE_NOTE", payload: id})
       fetch("http://127.0.0.1:9999/notes/" + id,{method: "DELETE"});
    }

    render(){
        const {title, text} = this.props;
        return (
            <Consumer>
                {
                    value=> {
                        const {dispatch} = value;
                        return (
                            <div class="card col-3 ml-5 mt-3">
                                <div class="card-body">
                                    <h3> {title} </h3>
                                    <p> {text}</p>
                                    <button class="btn btn-success card-link" 
                                    onClick={this.deleteNote.bind(this,dispatch)}>Delete</button>
                                </div>
                            </div>
                        )
                    }
                }
            </Consumer>
        )
    }
}
{% endhighlight %}



<b>react/components/cards.js</b>
{% highlight javascript %}

class NotesCard extends React.Component {


    render(){
        return (
            <Consumer>
                {
                    value=>{
                        const {notes}= value;
                        return (
                            <div className="row mt-5">
                                {
                                    notes.map(data=> (
                                        <Note key={data.id}  
                                            id={data.id}
                                            title= {data.note_name} 
                                            text={data.note} 
                                        />       
                                    ))
                                }
                            </div>
                        )
                    }
                }
            </Consumer>
        )
    }
}
{% endhighlight %}



<b>react/components/modal.js</b>
{% highlight javascript %}



class Modal extends React.Component {

    addNote= (dispatch,e)=>{

        const createdId = Math.random()+1;
        const noteName = this.noteName.value;
        const note = this.note.value;

        if(noteName == "" && note == "")
            return false;
        

        const requestOptions = {
            method: "POST",
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                id: createdId,
                note_name : noteName,
                note: note
            })
        }
        fetch("http://127.0.0.1:9999/notes", requestOptions)
        .catch(error => {
            console.error("hata oluştu");
        });

        dispatch({ type:"ADD_NOTE", 
            payload:{id : createdId, note_name : noteName, note: note}});   
    }
    render() {
        return (
            <Consumer>
                {
                    value=> {

                        const {dispatch} = value;

                        return (
                            <div class="modal fade " id="noteModal" tabIndex="-1" role="dialog" aria-labelledby="noteModalLabel" aria-hidden="true">
                            <div class="modal-dialog" role="document">
                                <div class="modal-content">
                                    <div class="modal-header bg-success">
                                        <h5 class="modal-title" id="exampleModalLabel">Creating Your Note</h5>
                                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                        <span aria-hidden="true">&times;</span>
                                        </button>
                                    </div>
                                    <div class="modal-body">
                                        <form>
                                            <div class="form-group">
                                                <label htmlFor="noteNameInput">Name</label>
                                                <input type="text" class="form-control" id="noteNameInput" 
                                                   ref={(c) => this.noteName = c}
                                                   aria-describedby="noteName" />
                                            </div>
                
                                            <div class="form-group">
                                                <label htmlFor="noteInput">Note</label>
                                                <textarea class="form-control" id="noteInput" rows="3"
                                                    ref={(c) => this.note = c}
                                                ></textarea>
                                            </div>
                                        </form>
                                    </div>
                                    <div class="modal-footer">
                                        <button type="Submit" 
                                            onClick = {this.addNote.bind(this,dispatch)}
                                        class="btn btn-primary" data-dismiss="modal">Save</button>
                                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                        )
                        
                    }
                }
            </Consumer>
        )
        
    }
}

{% endhighlight %}

<b>react/components/navbar.js</b>
{% highlight javascript %}



class Navbar extends React.Component {
    render() {
        return (
            <nav class="navbar bg-success navbar-expand-lg navbar-light">
            <a class="navbar-brand" href="#">Notepad</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item">
                        <button type="button" class="btn" data-toggle="modal" data-target="#noteModal">
                            Create Note
                        </button>    
                    </li>
                </ul>
            </div>
        </nav>
        )
    }
}
{% endhighlight %}



<b>db.json</b>
{% highlight json %}

{
  "notes": [
    {
      "id": 1.5683959873319369,
      "note_name": "İlk Not",
      "note": "Bu benim ilk notum."
    }
  ]
}

{% endhighlight %}




<b> Json Server'ı çalıştırma</b>
{% highlight javascript %}
json-server.cmd --watch .\db.json --port 9999
{% endhighlight %}


<b> Deno çalıştırma </b>
{% highlight javascript %}
deno run --allow-read --allow-net --importmap=importmap.json --unstable mod.js
{% endhighlight %}
