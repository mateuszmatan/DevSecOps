from openai import AzureOpenAI
import streamlit as st
from streamlit_modal import Modal
import sparql_dataframe
from random import randrange
from PIL import Image
import configuration
#TOGETHER_API_KEY =''
from pygwalker.api.streamlit import get_streamlit_html


gpt_4o_rk = configuration.gpt_4o_rk
gpt_35_turbo = configuration.gpt_35_turbo

key = configuration.key

DN_Mesage = ["I'm sorry I don't have knowledge to answer the question.","That’s beyond my realm of knowledge."]

client = AzureOpenAI(
  azure_endpoint= gpt_35_turbo,
  api_key=key,
  api_version=configuration.api_version
)

img = Image.open(configuration.logo_flag)
st.set_page_config(page_title='Makerspace Metadata Assistant',page_icon=img,layout="wide")
st.set_option('client.showErrorDetails', False)

#st.image(configuration.logo_path)
st.logo(configuration.logo_path)
st.header('')

st.markdown(
    """
        <style>
        [data-testid="stAppViewContainer"] {
        background-image: url("https://images.unsplash.com/photo-1644088379091-d574269d422f?q=80&w=1993&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D");
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        background-attachment: local;
        }
        .st-emotion-cache-4oy321 {
            border-style:solid;
            border-color: #3504b5;
            background-color: white;
        }
        .st-emotion-cache-1c7y2kd {
            border-style: solid;
            border-color: #3504b5;
            background-color: rgb(216,216,216);
        }
        .st-emotion-cache-uzeiqp {
            color: black;
        }
        .st-emotion-cache-12fmjuu {
            background-color: rgb(10,0,35); 
            height: 4rem;
        }
        .st-emotion-cache-qdbtli {
            background-color: rgb(10,0,35); 
            padding-top: 1.5rem;
            padding-bottom: 1.5rem;
        }
        .st-emotion-cache-1whx7iy {
            color:black;
        }
        .st-emotion-cache-1wmy9hl {
            padding-right: 5rem;
        }
    </style>
    """, unsafe_allow_html=True)

#azure_deployment_name = "mmdeptest"

#client = OpenAI(api_key=TOGETHER_API_KEY,#
#  base_url='https://api.together.xyz/v1'
#)

#source = data.cars()

def find_between( s, first, last ):
    try:
        start = s.index( first ) + len( first )
        end = s.index( last, start )
        return s[start:end]
    except ValueError:
        return ""

#init_prompt = """You are a great assistant at vega-lite visualization creation. No matter what the user ask, you should always response with a valid vega-lite specification in JSON.
#
#            You should create the vega-lite specification based on user's query.
#
#            Besides, Here are some requirements:
#            1. Do not contain the key called 'data' in vega-lite specification.
#            2. If the user ask many times, you should generate the specification based on the previous context.
#            3. You should consider to aggregate the field if it is quantitative and the chart has a mark type of react, bar, line, area or arc.
#            4. Consider to use bin for field if it is a chart like heatmap or histogram.
#            5. The only available fields that user can ask for and their types are: Horsepower quantitative ,   Name nominal, Origin  nominal
#            """

init_prompt = """
You are a great assistant at SPARQL query creation.
No matter what the user ask, you should always response with a valid SPARQL 1.1 specification.
You should create the SPARQL query based on user's query. The PREFIX is \<http://www.bbh.com/rolx#>.
Be precise and short with answers. Data property of User is 'userName'. Data property of Report is 'reportName'. Data property of report type is 'reportType'. Data property of Fund is 'fundName'
Data property of GAV is 'gavValue'. Data property of Company is 'companyName'. Data property of capital raised  is 'capitalRaisedValue'. Data property of Execution is 'executionStatus' and 'executionDuration' and 'executionDate' Report 'hasExecution'. Answer with ONLY one SPARQL 1.1 query.
"""



modal = Modal(
"Data Visualization"
,
key
=
"data-visualization"
,

padding
=20, # default value
max_width
=744 # default value
)

# Set a default model
if "openai_model" not in st.session_state:
    st.session_state["openai_model"] = "codellama/CodeLlama-13b-Instruct-hf"
    st.session_state["output_length"] = 200
    st.session_state["temperature"] = 0.43

# Initialize chat history
if "messages" not in st.session_state:
    st.session_state.messages = []
    st.session_state.messages_ui = []
    st.session_state.messages.append({"role": "user", "content": init_prompt})
    st.session_state.messages_ui.append({"role": "assistant", "content": 'Hello, I am your **Makerspace Metadata Assistant**. Ask me about **reporting** aspects in our firm and **investment market** information :)'})

for message in st.session_state.messages_ui:
    with st.chat_message(message["role"]):
        if message.get("type") == "table":
            st.dataframe(message["content"])         
        else:
            st.markdown(message["content"])        


# Accept user input
if prompt := st.chat_input("Type your question here"):
    # Add user message to chat history
    st.session_state.messages.append({"role": "user", "content": prompt})
    st.session_state.messages_ui.append({"role": "user", "content": prompt})
    # Display user message in chat message container
    with st.chat_message("user"):
        st.markdown(prompt)

    # Display assistant response in chat message container
    with st.chat_message("assistant"):
        #try:        
            chat_completion = client.chat.completions.create(
                #model=st.session_state["openai_model"],
                model=configuration.model,
                temperature=st.session_state["temperature"],
                max_tokens=st.session_state["output_length"],
                messages=[                
                    {"role": m["role"], "content": m["content"]}
                    for m in st.session_state.messages
                ],
                stream=False,
            )



            response = chat_completion.choices[0].message.content

            print(response)
            #response = st.write(chat_completion.choices[0].message.content)        
            #chart = find_between(response,"```","```")

            #st.vega_lite_chart(source, json.loads(chat_completion.choices[0].message.content), theme="streamlit", use_container_width=True)


            #q=urllib.parse.quote_plus(find_between(response,"```","```"))
            #q=find_between(response,"```","```")
            q=response
            print(q)

            endpoint = configuration.endpoint_url

            context_data_df = sparql_dataframe.get(endpoint, q.encode('utf-8'))

            #url = "http://127.0.0.1:8080/sparql?query="
            #x = requests.get(url+q)        


            #contextdata = x.json()

            #print(contextdata)

            #context_data_columname = contextdata['head']['vars'][0]


            #contextdata_list = []

            #for row in contextdata['results']['bindings']:
                #contextdata_list.append(row[context_data_columname]['value'])

            #context_data_df = pd.DataFrame(contextdata_list, columns = contextdata['head']['vars'])

            #print(context_data_df)

            tab1, tab2 = st.tabs(["🗃 Data", "📈 Chart"])           

            html = get_streamlit_html(context_data_df,  use_kernel_calc=True)      
            tab2._html(html, height=1000, scrolling=True)

            tab1.dataframe(context_data_df)

            
            #pyg_app = StreamlitRenderer(context_data_df)
 
            #st.write(pyg_app.explorer())
            
            #AgGrid(context_data_df)

            #st.json(x.json())


            #st.write("I'm sorry I was not able to answer.")
            #st.markdown(context_data_df)
            st.session_state.messages_ui.append({"role": "assistant","type": "table" ,"content": context_data_df})
            st.session_state.messages.append({"role": "assistant", "content": response})
        #am.write("I'm sorry I don't have knowleadge to asnwer the question.") 
        #except Exception as e:   
        #    print(e)
        #    m = DN_Mesage[randrange(2)]
            st.session_state.messages_ui.append({"role": "assistant", "content": m})  
            st.markdown(m)
            
