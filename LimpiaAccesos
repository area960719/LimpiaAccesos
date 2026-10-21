using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Reflection;
using System.Windows.Forms;

namespace LimpiaAccesos
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.Run(new FormPrincipal());
        }
    }

    public class FormPrincipal : Form
    {
        const string T_LNK = "Acceso directo falso";
        const string T_EXE = "Archivo oculto sospechoso";
        const string T_OCULTO = "Oculto por virus (restaurar)";

        ComboBox cmbRuta;
        Button btnActualizar, btnExaminar, btnEscanear, btnRapido, btnMarcar, btnLimpiar;
        CheckBox chkSub, chkUSB;
        ListView lista;
        Label lblEstado;

        // true = memoria USB: se restaura todo lo oculto. false = PC: solo lo que un acceso directo falso esta tapando.
        bool modoUSB;

        static readonly string[] Protegidos =
        {
            "system volume information", "$recycle.bin", "recycler",
            "recycled", "found.000", "desktop.ini", "thumbs.db"
        };

        // Carpetas del sistema que se saltan cuando se escanea una unidad de la PC (solo en la raiz)
        static readonly string[] CarpetasSistema =
        {
            "windows", "program files", "program files (x86)", "programdata",
            "$windows.~bt", "$windows.~ws", "recovery", "perflogs", "msocache"
        };

        static readonly string[] ExtPeligrosas =
        {
            ".exe", ".scr", ".pif", ".com", ".bat", ".cmd",
            ".vbs", ".vbe", ".js", ".jse", ".wsf", ".wsh"
        };

        static readonly string[] PalabrasLnk =
        {
            "cmd.exe", "wscript", "cscript", "powershell", "mshta",
            "rundll32", ".vbs", ".vbe", ".bat", ".cmd", ".wsf", ".js"
        };

        public FormPrincipal()
        {
            Text = "Limpiador de accesos directos (PC y USB)";
            Size = new Size(860, 580);
            MinimumSize = new Size(740, 460);
            StartPosition = FormStartPosition.CenterScreen;

            Label lbl = new Label();
            lbl.Text = "Unidad o carpeta:";
            lbl.Location = new Point(12, 16);
            lbl.AutoSize = true;
            Controls.Add(lbl);

            cmbRuta = new ComboBox();
            cmbRuta.Location = new Point(120, 12);
            cmbRuta.Width = 460;
            cmbRuta.Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right;
            Controls.Add(cmbRuta);

            btnActualizar = new Button();
            btnActualizar.Text = "Actualizar";
            btnActualizar.Location = new Point(590, 10);
            btnActualizar.Size = new Size(80, 26);
            btnActualizar.Anchor = AnchorStyles.Top | AnchorStyles.Right;
            btnActualizar.Click += delegate { CargarUnidades(); };
            Controls.Add(btnActualizar);

            btnExaminar = new Button();
            btnExaminar.Text = "Examinar...";
            btnExaminar.Location = new Point(676, 10);
            btnExaminar.Size = new Size(90, 26);
            btnExaminar.Anchor = AnchorStyles.Top | AnchorStyles.Right;
            btnExaminar.Click += delegate { Examinar(); };
            Controls.Add(btnExaminar);

            chkSub = new CheckBox();
            chkSub.Text = "Buscar tambien dentro de las carpetas";
            chkSub.Location = new Point(12, 44);
            chkSub.AutoSize = true;
            chkSub.Checked = true;
            Controls.Add(chkSub);

            chkUSB = new CheckBox();
            chkUSB.Text = "Tratar como memoria USB (restaurar todo lo oculto)";
            chkUSB.Location = new Point(300, 44);
            chkUSB.AutoSize = true;
            Controls.Add(chkUSB);

            btnEscanear = new Button();
            btnEscanear.Text = "1. Escanear unidad/carpeta";
            btnEscanear.Location = new Point(12, 72);
            btnEscanear.Size = new Size(180, 32);
            btnEscanear.Click += delegate { Escanear(); };
            Controls.Add(btnEscanear);

            btnRapido = new Button();
            btnRapido.Text = "Escaneo rapido de PC";
            btnRapido.Location = new Point(200, 72);
            btnRapido.Size = new Size(150, 32);
            btnRapido.Click += delegate { EscaneoRapido(); };
            Controls.Add(btnRapido);

            btnMarcar = new Button();
            btnMarcar.Text = "Marcar / desmarcar todo";
            btnMarcar.Location = new Point(358, 72);
            btnMarcar.Size = new Size(150, 32);
            btnMarcar.Click += delegate { MarcarTodo(); };
            Controls.Add(btnMarcar);

            btnLimpiar = new Button();
            btnLimpiar.Text = "2. Limpiar marcados";
            btnLimpiar.Location = new Point(516, 72);
            btnLimpiar.Size = new Size(150, 32);
            btnLimpiar.Click += delegate { Limpiar(); };
            Controls.Add(btnLimpiar);

            lista = new ListView();
            lista.Location = new Point(12, 112);
            lista.Size = new Size(820, 390);
            lista.Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right;
            lista.View = View.Details;
            lista.CheckBoxes = true;
            lista.FullRowSelect = true;
            lista.GridLines = true;
            lista.Columns.Add("Ruta", 400);
            lista.Columns.Add("Tipo", 170);
            lista.Columns.Add("Detalle", 230);
            Controls.Add(lista);

            lblEstado = new Label();
            lblEstado.Text = "Elige una unidad y pulsa Escanear, o usa el Escaneo rapido de PC.";
            lblEstado.Location = new Point(12, 514);
            lblEstado.AutoSize = true;
            lblEstado.Anchor = AnchorStyles.Bottom | AnchorStyles.Left;
            Controls.Add(lblEstado);

            CargarUnidades();
        }

        void CargarUnidades()
        {
            cmbRuta.Items.Clear();
            DriveInfo[] unidades = DriveInfo.GetDrives();

            // Primero las memorias USB, luego el resto
            foreach (DriveInfo d in unidades)
            {
                try { if (d.IsReady && d.DriveType == DriveType.Removable) cmbRuta.Items.Add(d.Name); }
                catch { }
            }
            foreach (DriveInfo d in unidades)
            {
                try { if (d.IsReady && d.DriveType != DriveType.Removable && d.DriveType != DriveType.CDRom) cmbRuta.Items.Add(d.Name); }
                catch { }
            }

            if (cmbRuta.Items.Count > 0) cmbRuta.SelectedIndex = 0;
            else cmbRuta.Text = "";
        }

        void Examinar()
        {
            using (FolderBrowserDialog fb = new FolderBrowserDialog())
            {
                fb.Description = "Elige la unidad o la carpeta a limpiar";
                if (fb.ShowDialog() == DialogResult.OK)
                    cmbRuta.Text = fb.SelectedPath;
            }
        }

        void MarcarTodo()
        {
            bool todosMarcados = true;
            foreach (ListViewItem it in lista.Items)
                if (!it.Checked) { todosMarcados = false; break; }
            foreach (ListViewItem it in lista.Items)
                it.Checked = !todosMarcados;
        }

        static bool EsUSB(string ruta)
        {
            try { return new DriveInfo(Path.GetPathRoot(ruta)).DriveType == DriveType.Removable; }
            catch { return false; }
        }

        void Escanear()
        {
            string raiz = cmbRuta.Text.Trim();
            if (!Directory.Exists(raiz))
            {
                MessageBox.Show("Esa unidad o carpeta no existe.", "Limpiador");
                return;
            }
            Iniciar(new List<string> { raiz }, EsUSB(raiz) || chkUSB.Checked);
        }

        void EscaneoRapido()
        {
            Iniciar(RutasRapidas(), false);
        }

        static List<string> RutasRapidas()
        {
            string perfil = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
            string[] candidatas =
            {
                Environment.GetFolderPath(Environment.SpecialFolder.DesktopDirectory),
                Environment.GetFolderPath(Environment.SpecialFolder.CommonDesktopDirectory),
                Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
                Path.Combine(perfil, "Downloads"),
                Environment.GetFolderPath(Environment.SpecialFolder.Startup),
                Environment.GetFolderPath(Environment.SpecialFolder.CommonStartup)
            };

            List<string> rutas = new List<string>();
            foreach (string c in candidatas)
            {
                if (string.IsNullOrEmpty(c) || !Directory.Exists(c)) continue;
                if (!rutas.Contains(c)) rutas.Add(c);
            }
            return rutas;
        }

        void Iniciar(List<string> rutas, bool usb)
        {
            modoUSB = usb;
            lista.Items.Clear();
            Cursor = Cursors.WaitCursor;
            lblEstado.Text = "Escaneando...";
            try
            {
                foreach (string r in rutas)
                    Recorrer(r, chkSub.Checked);
            }
            finally { Cursor = Cursors.Default; }

            lblEstado.Text = "Escaneo terminado (modo " + (usb ? "USB" : "PC") + "): "
                + lista.Items.Count + " elementos. Revisa la lista y pulsa Limpiar.";
        }

        void Recorrer(string dir, bool recursivo)
        {
            FileSystemInfo[] entradas;
            try { entradas = new DirectoryInfo(dir).GetFileSystemInfos(); }
            catch { return; }

            Application.DoEvents();

            bool enRaiz = string.Equals(Path.GetPathRoot(dir), dir, StringComparison.OrdinalIgnoreCase);

            // Nombres de elementos ocultos (que no son .lnk) que un acceso directo falso podria estar tapando
            HashSet<string> ocultos = new HashSet<string>();
            foreach (FileSystemInfo e in entradas)
            {
                if ((e.Attributes & FileAttributes.Hidden) == 0) continue;
                if (e.Extension.ToLowerInvariant() == ".lnk") continue;
                ocultos.Add(e.Name.ToLowerInvariant());
                ocultos.Add(Path.GetFileNameWithoutExtension(e.Name).ToLowerInvariant());
            }

            // Nombres tapados por un acceso directo del mismo nombre
            HashSet<string> tapados = new HashSet<string>();
            foreach (FileSystemInfo e in entradas)
            {
                if (e.Extension.ToLowerInvariant() != ".lnk") continue;
                string b = Path.GetFileNameWithoutExtension(e.Name).ToLowerInvariant();
                if (ocultos.Contains(b)) tapados.Add(b);
            }

            foreach (FileSystemInfo e in entradas)
            {
                string nombre = e.Name.ToLowerInvariant();
                string ext = e.Extension.ToLowerInvariant();
                bool esDir = (e.Attributes & FileAttributes.Directory) != 0;
                bool oculto = (e.Attributes & FileAttributes.Hidden) != 0
                           || (e.Attributes & FileAttributes.System) != 0;

                if (Array.IndexOf(Protegidos, nombre) >= 0) continue;
                if ((e.Attributes & FileAttributes.ReparsePoint) != 0) continue;

                // En la PC no se toca el sistema ni AppData
                if (!modoUSB && esDir)
                {
                    if (enRaiz && Array.IndexOf(CarpetasSistema, nombre) >= 0) continue;
                    if (nombre == "appdata") continue;
                }

                if (!esDir && ext == ".lnk")
                {
                    string baseNombre = Path.GetFileNameWithoutExtension(e.Name).ToLowerInvariant();
                    string destino = LeerDestino(e.FullName);

                    bool ejecuta = false;
                    foreach (string p in PalabrasLnk)
                        if (destino.Contains(p)) { ejecuta = true; break; }
                    bool tapa = tapados.Contains(baseNombre);

                    if (tapa)
                    {
                        string m = "Tapa un elemento oculto con el mismo nombre";
                        if (ejecuta) m += " y ejecuta comandos";
                        Agregar(e.FullName, T_LNK, m, true);
                    }
                    else if (ejecuta)
                    {
                        string corto = destino.Length > 60 ? destino.Substring(0, 60) + "..." : destino;
                        // En USB se marca solo; en PC puede ser un acceso directo legitimo, asi que queda sin marcar
                        Agregar(e.FullName, T_LNK, (modoUSB ? "Ejecuta: " : "Ejecuta (revisar): ") + corto, modoUSB);
                    }
                }
                else if (oculto)
                {
                    if (nombre == "autorun.inf" || (!esDir && Array.IndexOf(ExtPeligrosas, ext) >= 0))
                    {
                        Agregar(e.FullName, T_EXE, "Ejecutable/script oculto (revisalo antes de borrar)", false);
                    }
                    else
                    {
                        string sinExt = Path.GetFileNameWithoutExtension(e.Name).ToLowerInvariant();
                        bool restaurar = modoUSB || tapados.Contains(nombre) || tapados.Contains(sinExt);
                        if (restaurar)
                            Agregar(e.FullName, T_OCULTO, esDir ? "Carpeta oculta" : "Archivo oculto", true);
                    }
                }

                if (esDir && recursivo) Recorrer(e.FullName, true);
            }
        }

        void Agregar(string ruta, string tipo, string detalle, bool marcado)
        {
            ListViewItem it = new ListViewItem(ruta);
            it.SubItems.Add(tipo);
            it.SubItems.Add(detalle);
            it.Checked = marcado;
            lista.Items.Add(it);
        }

        static string LeerDestino(string lnk)
        {
            try
            {
                Type t = Type.GetTypeFromProgID("WScript.Shell");
                object shell = Activator.CreateInstance(t);
                object acceso = t.InvokeMember("CreateShortcut", BindingFlags.InvokeMethod, null, shell, new object[] { lnk });
                Type ta = acceso.GetType();
                string destino = (string)ta.InvokeMember("TargetPath", BindingFlags.GetProperty, null, acceso, null);
                string args = (string)ta.InvokeMember("Arguments", BindingFlags.GetProperty, null, acceso, null);
                return ((destino ?? "") + " " + (args ?? "")).Trim().ToLowerInvariant();
            }
            catch { return ""; }
        }

        static void Registrar(string linea)
        {
            try
            {
                string log = Path.Combine(Application.StartupPath, "LimpiaAccesos_registro.txt");
                File.AppendAllText(log, DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss") + "  " + linea + Environment.NewLine);
            }
            catch { }
        }

        void Limpiar()
        {
            List<ListViewItem> marcados = new List<ListViewItem>();
            foreach (ListViewItem it in lista.Items)
                if (it.Checked) marcados.Add(it);

            if (marcados.Count == 0)
            {
                MessageBox.Show("No hay nada marcado.", "Limpiador");
                return;
            }

            DialogResult r = MessageBox.Show(
                "Se van a limpiar " + marcados.Count + " elementos:\n" +
                "- Los accesos directos y archivos sospechosos marcados se BORRAN (sin papelera).\n" +
                "- Los elementos ocultos se vuelven a mostrar.\n\nContinuar?",
                "Limpiador", MessageBoxButtons.YesNo, MessageBoxIcon.Question);
            if (r != DialogResult.Yes) return;

            int borrados = 0, restaurados = 0, errores = 0;
            Cursor = Cursors.WaitCursor;

            try
            {
                foreach (ListViewItem it in marcados)
                {
                    if (it.SubItems[1].Text == T_OCULTO) continue;
                    try
                    {
                        File.SetAttributes(it.Text, FileAttributes.Normal);
                        File.Delete(it.Text);
                        borrados++;
                        Registrar("BORRADO    " + it.Text);
                    }
                    catch { errores++; Registrar("ERROR      " + it.Text); }
                }

                foreach (ListViewItem it in marcados)
                {
                    if (it.SubItems[1].Text != T_OCULTO) continue;
                    try
                    {
                        FileAttributes a = File.GetAttributes(it.Text);
                        a &= ~FileAttributes.Hidden;
                        a &= ~FileAttributes.System;
                        File.SetAttributes(it.Text, a);
                        restaurados++;
                        Registrar("RESTAURADO " + it.Text);
                    }
                    catch { errores++; Registrar("ERROR      " + it.Text); }
                }
            }
            finally { Cursor = Cursors.Default; }

            Escanear();
            lblEstado.Text = "Listo: " + borrados + " borrados, " + restaurados + " restaurados, " + errores + " errores.";

            if (errores > 0)
                MessageBox.Show("Hubo " + errores + " errores. Puede ser falta de permisos (prueba abrir el programa como administrador) o una memoria protegida contra escritura.", "Limpiador");
        }
    }
}
