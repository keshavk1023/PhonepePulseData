# Importing necessary libraries
import pandas as pd
import mysql.connector as sql
import streamlit as st
import plotly.express as px
from streamlit_option_menu import option_menu
from PIL import Image

# Setting up page configuration
st.set_page_config(
    page_title="Phonepe Pulse Data Visualization",
    page_icon=None,
    layout="wide",
    initial_sidebar_state="expanded",
    menu_items={'About': """Data has been cloned from Phonepe Pulse Github Repository"""}
)

# Setting up sidebar header
st.sidebar.header(":Blue[**Welcome to the dashboard**]")

# Establishing connection to MySQL database
mydb = sql.connect(host="localhost",
                   user="root",
                   password="12212112",
                   database="phonepe"
                  )
mycursor = mydb.cursor(buffered=True)

# Setting up background image
st.markdown(f""" <style>.stApp {{
                        background:url("https://wallpapercave.com/wp/TN2ZHuz.jpg");
                        background-size: cover}}
                     </style>""", unsafe_allow_html=True)

# Creating sidebar option menu
with st.sidebar:
    selected = option_menu("Menu", ["Home", "Top Charts", "Explore Data", "About"],
                           icons=["house", "graph-up-arrow", "bar-chart-line", "exclamation-circle"],
                           menu_icon="menu-button-wide",
                           default_index=0,
                           styles={"nav-link": {"font-size": "15px", "text-align": "center", "margin": "7px",
                                                "--hover-color": "#6F36AD"},
                                   "nav-link-selected": {"background-color": "#6F36AD"}})

# Function to create pie chart
def piechart(value, name, dataframe, h_data):
    st.markdown(f"### :Blue[{name}]")
    df = dataframe
    fig = px.pie(df, values=value,
                 names=name,
                 title='Top 10',
                 color_discrete_sequence=px.colors.sequential.Agsunset,
                 hover_data=[h_data],
                 labels={'Transactions_Count': 'Transactions_Count'})
    fig.update_traces(textposition='inside', textinfo='percent+label')
    st.plotly_chart(fig, use_container_width=True)

# Function to create bar graph
def bargraph(x_value, y_value, colour, dataframe):
    st.markdown(f"### :Blue[{y_value}]")
    df = dataframe
    fig = px.bar(df, x=x_value,
                 y=y_value,
                 orientation='v',
                 color=colour,
                 color_continuous_scale=px.colors.sequential.Agsunset)
    st.plotly_chart(fig, use_container_width=True)

# Function to create geo map
def geomap(location, colour, dataframe):
    df1 = dataframe
    df2 = pd.read_csv('Statenames.csv')  # Assuming 'Statenames.csv' contains state names mapping
    df1.State = df2

    fig = px.choropleth(df1, geojson="https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson",
                        featureidkey='properties.ST_NM',
                        locations=location,
                        color=colour,
                        color_continuous_scale='sunset')

    fig.update_geos(fitbounds="locations", visible=False)
    st.plotly_chart(fig, use_container_width=True)

# Handling menu option "Home"
if selected == "Home":
    st.markdown("# :Blue[Data Visualization and Exploration]")
    st.markdown("### :Blue[A User-Friendly Tool Using Streamlit and Plotly]")
    col1, col2 = st.columns([3, 2], gap="medium")
    with col1:
        st.write(" ")
        st.write(" ")
        st.markdown("## :Blue[Overview :]")
        st.markdown("### In this streamlit web app you can visualize the phonepe pulse data and gain lot of insights on transactions, number of users, top 10 state, district, pincode and which brand has most number of users and so on. Bar charts, Pie charts and Geo map visualization are used to get some insights.")
    with col2:
        st.write("")
        st.write("")
        st.markdown("# :Blue[Technologies used :]")
   
        st.info(
        """
        - GITHUB CLONING.
        - PYTHON.
        - PANDAS.
        - MYSQL.
        - STREAMLIT.
        - PLOTLY.                
        """ )

# Handling menu option "Top Charts"
if selected == "Top Charts":
    # UI setup
    st.markdown("# TOP CHARTS")
    Type = st.sidebar.selectbox("**Type**", ("Transactions", "Users"))
    column1, column2 = st.columns([1, 1.5], gap="large")
    with column2:
        st.markdown("")
        st.markdown("")
        st.markdown("")
        a, b = st.columns([2, 2])
        years = [2018, 2019, 2020, 2021, 2022]
        Quarters = [1, 2, 3, 4]
        with a:
            Year = st.selectbox("select year", options=years)
        with b:
            Quarter = st.selectbox("select quater", options=Quarters)     
    
    with column1:
        st.info(
            """
            #### From this menu we can get insights like :
            - Overall ranking on a particular Year and Quarter.
            - Top 10 State, District, Pincode based on Total number of transaction and Total amount spent on phonepe.
            - Top 10 State, District, Pincode based on Total phonepe users and their app opening frequency.
            - Top 10 mobile brands and its percentage based on the how many people use phonepe.
            """, icon="🔍"
        )
        
    # Handling Type selection
    if Type == "Transactions":
        col1, col2, col3 = st.columns([1, 1, 1], gap="small")
        
        with col1:
            mycursor.execute(f"select state, sum(Transaction_count) as Total_Transactions_Count, sum(Transaction_amount) as Total from agg_transaction where year = {Year} and quarter = {Quarter} group by state order by Total desc limit 10")
            piechart("Total_Amount", "State", pd.DataFrame(mycursor.fetchall(), columns=["State", 'Transactions_Count', 'Total_Amount']), 'Transactions_Count')
            
        with col2:
            mycursor.execute(f"select district , sum(Count) as Total_Count, sum(Amount) as Total from map_trans where year = {Year} and quarter = {Quarter} group by district order by Total desc limit 10")
            piechart("Total_Amount", "District", pd.DataFrame(mycursor.fetchall(), columns=["District", 'Transactions_Count', 'Total_Amount']), 'Transactions_Count')
            
        with col3:
            mycursor.execute(f"select pincode, sum(Transaction_count) as Total_Transactions_Count, sum(Transaction_amount) as Total from top_trans where year = {Year} and quarter = {Quarter} group by pincode order by Total desc limit 10")
            piechart("Total_Amount", "Pincode", pd.DataFrame(mycursor.fetchall(), columns=["Pincode", 'Transactions_Count', 'Total_Amount']), 'Transactions_Count')
            
    # Handling Type "Users"
    if Type == "Users":
        col1, col2, col3, col4 = st.columns([2, 2, 2, 2], gap="small")
        
        with col1:
            if Year == 2022 and Quarter in [2, 3, 4]:
                st.markdown("#### Sorry No Data to Display for 2022 Qtr 2,3,4")
            else:
                mycursor.execute(f"select brands, sum(count) as Total_Count, avg(percentage)*100 as Avg_Percentage from agg_users where year = {Year} and quarter = {Quarter} group by brands order by Total_Count desc limit 10")
                bargraph("Total_Users", "Brand", 'Total_Users', pd.DataFrame(mycursor.fetchall(), columns=['Brand', 'Total_Users', 'Avg_Percentage'])) 

        with col2:
            mycursor.execute(f"select district, sum(RegisteredUser) as Total_Users, sum(AppOpens) as Total_Appopens from map_user where year = {Year} and quarter = {Quarter} group by district order by Total_Users desc limit 10")
            bargraph("Total_Users", "District", 'Total_Users', pd.DataFrame(mycursor.fetchall(), columns=['District', 'Total_Users', 'Total_Appopens']))
              
        with col3:
            mycursor.execute(f"select Pincode, sum(Registered_users) as Total_Users from top_user where year = {Year} and quarter = {Quarter} group by Pincode order by Total_Users desc limit 10")
            piechart('Total_Users', 'Pincode', pd.DataFrame(mycursor.fetchall(), columns=['Pincode', 'Total_Users']), 'Total_Users')
           
        with col4:
            mycursor.execute(f"select state, sum(RegisteredUser) as Total_Users, sum(AppOpens) as Total_Appopens from map_user where year = {Year} and quarter = {Quarter} group by state order by Total_Users desc limit 10")
            piechart('Total_Users', 'State', pd.DataFrame(mycursor.fetchall(), columns=['State', 'Total_Users', 'Total_Appopens']), 'Total_Appopens')

# Handling menu option "Explore Data"
if selected == "Explore Data":
    Year = st.sidebar.slider("**Year**", min_value=2018, max_value=2022, label_visibility="hidden")
    Quarter = st.sidebar.slider("Quarter", min_value=1, max_value=4, label_visibility="hidden")
    Type = st.sidebar.selectbox("**Type**", ("Transactions", "Users"))
    col1, col2 = st.columns(2)
    
    # Handling Type "Transactions"
    if Type == "Transactions":
        
        with col1:
            st.markdown("## :Blue[Overall State Data - Transactions Amount]")
            mycursor.execute(f"select state, sum(count) as Total_Transactions, sum(amount) as Total_amount from map_trans where year = {Year} and quarter = {Quarter} group by state order by state")
            geomap("State", "Total_amount", pd.DataFrame(mycursor.fetchall(), columns=['State', 'Total_Transactions', 'Total_amount']))
                
        with col2: 
            st.markdown("## :Blue[Overall State Data - Transactions Count]")
            mycursor.execute(f"select state, sum(count) as Total_Transactions, sum(amount) as Total_amount from map_trans where year = {Year} and quarter = {Quarter} group by state order by state")
            geomap("State", "Total_Transactions", pd.DataFrame(mycursor.fetchall(), columns=['State', 'Total_Transactions', 'Total_amount']))
            
        # Bar chart for top payment type
        st.markdown("## :Blue[Top Payment Type]")
        mycursor.execute(f"select Transaction_type, sum(Transaction_count) as Total_Transactions, sum(Transaction_amount) as Total_amount from agg_transaction where year= {Year} and quarter = {Quarter} group by transaction_type order by Transaction_type")
        bargraph("Transaction_type", "Total_Transactions", 'Total_amount', pd.DataFrame(mycursor.fetchall(), columns=['Transaction_type', 'Total_Transactions', 'Total_amount']))
        
        # Additional exploration option
        st.markdown("# ")
        st.markdown("# ")
        st.markdown("# ")
        st.markdown("## :Blue[Select any State to explore more]")
        selected_state = st.selectbox("",
                                      ('ndaman-&-nicobar-islands','andhra-pradesh','arunachal-pradesh','assam','bihar',
                                       'chandigarh','chhattisgarh','dadra-&-nagar-haveli-&-daman-&-diu','delhi','goa','gujarat','haryana',
                                       'himachal-pradesh','jammu-&-kashmir','jharkhand','karnataka','kerala','ladakh','lakshadweep',
                                       'madhya-pradesh','maharashtra','manipur','meghalaya','mizoram',
                                       'nagaland','odisha','puducherry','punjab','rajasthan','sikkim',
                                       'tamil-nadu','telangana','tripura','uttar-pradesh','uttarakhand','west-bengal'), index=30)
         
        mycursor.execute(f"select State, District,year,quarter, sum(count) as Total_Transactions, sum(amount) as Total_amount from map_trans where year = {Year} and quarter = {Quarter} and State = '{selected_state}' group by State, District,year,quarter order by state,district")
        bargraph("District", "Total_Transactions", 'Total_amount', pd.DataFrame(mycursor.fetchall(), columns=['State','District','Year','Quarter','Total_Transactions','Total_amount']))
       
    # Handling Type "Users"
    if Type == "Users":
        
        st.markdown("## :Blue[Overall State Data - User App opening frequency]")
        mycursor.execute(f"select state, sum(RegisteredUser) as Total_Users, sum(AppOpens) as Total_Appopens from map_user where year = Year and quarter = Quarter group by state order by state;")
        geomap('State', 'Total_Appopens', pd.DataFrame(mycursor.fetchall(), columns=['State', 'Total_Users', 'Total_Appopens']))
           
        st.markdown("## :Blue[Select any State to explore more]")
        selected_state = st.selectbox("",
                                      ('andaman-&-nicobar-islands','andhra-pradesh','arunachal-pradesh','assam','bihar',
                                       'chandigarh','chhattisgarh','dadra-&-nagar-haveli-&-daman-&-diu','delhi','goa','gujarat','haryana',
                                       'himachal-pradesh','jammu-&-kashmir','jharkhand','karnataka','kerala','ladakh','lakshadweep',
                                       'madhya-pradesh','maharashtra','manipur','meghalaya','mizoram',
                                       'nagaland','odisha','puducherry','punjab','rajasthan','sikkim',
                                       'tamil-nadu','telangana','tripura','uttar-pradesh','uttarakhand','west-bengal'), index=30)
        
        mycursor.execute(f"select State,year,quarter,District,sum(RegisteredUser) as Total_Users, sum(AppOpens) as Total_Appopens from map_user where year = {Year} and quarter = {Quarter} and state = '{selected_state}' group by State, District,year,quarter order by state,district")
        bargraph("District", "Total_Users", 'Total_Users', pd.DataFrame(mycursor.fetchall(), columns=['State','year', 'quarter', 'District', 'Total_Users','Total_Appopens']))

# Handling menu option "About"
if selected == "About":
    col1, col2 = st.columns([3, 3], gap="medium")
    with col1:
        st.write(" ")
        st.markdown("## :Blue[About PhonePe Pulse:] ")
        st.write("#### BENGALURU, India, On Sept. 3, 2021 PhonePe, India's leading fintech platform, announced the launch of PhonePe Pulse, India's first interactive website with data, insights and trends on digital payments in the country. The PhonePe Pulse website showcases more than 2000+ Crore transactions by consumers on an interactive map of India. With  over 45% market share, PhonePe's data is representative of the country's digital payment habits.")
        st.write("##### The insights on the website and in the report have been drawn from two key sources - the entirety of PhonePe's transaction data combined with merchant and customer interviews. The report is available as a free download on the PhonePe Pulse website and GitHub.")
               
    with col2:
        st.write(" ")
        st.write(" ")
        st.write(" ")
        st.write(" ")
        st.write(" ")
        st.write(" ")
        st.image("PhonePe.png")

    st.markdown("## :Blue[About PhonePe:] ")
    st.write("### PhonePe is India's leading fintech platform with over 300 million registered users. Using PhonePe, users can send and receive money, recharge mobile, DTH, pay at stores, make utility payments, buy gold and make investments. PhonePe forayed into financial services in 2017 with the launch of Gold providing users with a safe and convenient option to buy 24-karat gold securely on its platform. PhonePe has since launched several Mutual Funds and Insurance products like tax-saving funds, liquid funds, international travel insurance and Corona Care, a dedicated insurance product for the COVID-19 pandemic among others. PhonePe also launched its Switch platform in 2018, and today its customers can place orders on over 600 apps directly from within the PhonePe mobile app. PhonePe is accepted at 20+ million merchant outlets across Bharat")
    st.write("# **:Blue[My Project GitHub link]** ⬇️")
    st.write("#### https://github.com/keshavk1023/PhonepePulse")
    st.write("# **:Blue[content source]** ⬇️")
    st.write("#### https://www.phonepe.com/press/phonepe-launches-the-pulse-of-digital-payments-indias-first-interactive-geospatial-website/")
