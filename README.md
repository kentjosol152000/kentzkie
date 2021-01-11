# kentzkie
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Net;
using System.Net.Sockets;
using System.IO;


namespace clientserver
{
    public partial class Form1 : Form
    {

        private TcpClient client;
        public StreamReader STR;
        public StreamWriter STW;
        public string recieve;
        public string TextToSend;
        public Form1()
        {
            InitializeComponent();
            IPAddress[] localIP = Dns.GetHostAddresses(Dns.GetHostName());
            foreach (IPAddress address in localIP)
            {
                if (address.AddressFamily == AddressFamily.InterNetwork)
                {
                    serverIPtextBox.Text = address.ToString();
                }
            }
            
        }

        private void connectbutton_Click(object sender, EventArgs e)
        {
            client = new TcpClient();
            IPEndPoint IpEnd = new IPEndPoint(IPAddress.Parse(clientIPtextBox.Text), int.Parse(clientporttextBox.Text));
            try
            {
                client.Connect(IpEnd);
                if (client.Connected)
                {
                    listBox1.Items.Add("Me and Client Connected" + "\n");
                    STR = new StreamReader(client.GetStream());
                    STW = new StreamWriter(client.GetStream());
                    STW.AutoFlush = true;
                    backgroundWorker1.RunWorkerAsync();
                    backgroundWorker2.WorkerSupportsCancellation = true;
                }
                groupBox1.Visible = false;
                groupBox2.Visible = false;
                listBox1.Visible = true;
                MessagetextBox.Visible = true;
                sendbutton.Visible = true;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message.ToString());
            }


           
        }

        private void backgroundWorker2_DoWork(object sender, DoWorkEventArgs e)
        {
            if (client.Connected)
            {
                STW.WriteLine(TextToSend);
                this.listBox1.Invoke(new MethodInvoker(delegate ()
                {
                    listBox1.Items.Add("Me: " + TextToSend + "\n");
                   // listBox1.Items.Add("Client: " + TextToSend + "\n");
                }));

            }
            else
            {
                MessageBox.Show("Sending Failed");
            }
            backgroundWorker2.CancelAsync();
        }

        private void backgroundWorker1_DoWork(object sender, DoWorkEventArgs e)
        {
            while (client.Connected)
            {
                try
                {
                    recieve = STR.ReadLine();
                    this.listBox1.Invoke(new MethodInvoker(delegate ()
                    {
                       
                     
                        listBox1.Items.Add("Client: " + recieve + "\n");
                    }));
                    recieve = "";

                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message.ToString());
                }

            }
        }

        private void startbutton_Click(object sender, EventArgs e)
        {
            TcpListener listener = new TcpListener(IPAddress.Any, int.Parse(serverporttextBox.Text));
            listener.Start();
            client = listener.AcceptTcpClient();
            STR = new StreamReader(client.GetStream());
            STW = new StreamWriter(client.GetStream());
            STW.AutoFlush = true;
            backgroundWorker1.RunWorkerAsync();
            backgroundWorker2.WorkerSupportsCancellation = true;

            groupBox1.Visible = false;
            groupBox2.Visible = false;
            listBox1.Visible = true;
            MessagetextBox.Visible = true;
            sendbutton.Visible = true;
        }

        private void sendbutton_Click(object sender, EventArgs e)
        {
            if (MessagetextBox.Text != "")
            {
                TextToSend = MessagetextBox.Text;
                backgroundWorker2.RunWorkerAsync();
            }
            MessagetextBox.Text = "";
        }

        private void groupBox1_Enter(object sender, EventArgs e)
        {

        }

        private void listBox1_SelectedIndexChanged(object sender, EventArgs e)
        {

        }
    }
}
