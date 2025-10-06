# lab6
lab6
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Windows.Forms;
using lab6_01.Models;   
namespace lab6_01
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            try
            {
                Model1 context = new Model1();
                List<Faculty> listFaculties = context.Faculties.ToList();
                List<Student> listStudents = context.Students.ToList();

                FillFacultyCombobox(listFaculties);
                BindGrid(listStudents);
            }
            catch (Exception ex)
            {
                MessageBox.Show("Lỗi tải dữ liệu: " + ex.Message);
            }
        }

        private void FillFacultyCombobox(List<Faculty> listFaculties)
        {
            cmbFaculty.DataSource = listFaculties;
            cmbFaculty.DisplayMember = "FacultyName";
            cmbFaculty.ValueMember = "FacultyID";
        }

        
        private void BindGrid(List<Student> listStudents)
        {
            dgvStudent.Rows.Clear();

            foreach (var item in listStudents)
            {
                int index = dgvStudent.Rows.Add();
                dgvStudent.Rows[index].Cells[0].Value = item.StudentID;
                dgvStudent.Rows[index].Cells[1].Value = item.FullName;
                dgvStudent.Rows[index].Cells[2].Value = item.Faculty?.FacultyName;
                dgvStudent.Rows[index].Cells[3].Value = item.AverageScore;
            }
        }

        private void btnAdd_Click(object sender, EventArgs e)
        {
            if (string.IsNullOrEmpty(txtStudentID.Text) ||
                string.IsNullOrEmpty(txtFullName.Text) ||
                string.IsNullOrEmpty(txtAverageScore.Text))
            {
                MessageBox.Show("Vui lòng nhập đầy đủ thông tin!");
                return;
            }

            if (txtStudentID.Text.Length != 10)
            {
                MessageBox.Show("Mã số sinh viên phải có 10 kí tự!");
                return;
            }

            using (Model1 context = new Model1())
            {
                var existing = context.Students.FirstOrDefault(st => st.StudentID == txtStudentID.Text);
                if (existing != null)
                {
                    MessageBox.Show("Mã số sinh viên đã tồn tại!");
                    return;
                }

                var student = new Student()
                {
                    StudentID = txtStudentID.Text,
                    FullName = txtFullName.Text,
                    AverageScore = double.Parse(txtAverageScore.Text),
                    FacultyID = cmbFaculty.SelectedValue.ToString()
                };

                context.Students.Add(student);
                context.SaveChanges();
            }

            MessageBox.Show("Thêm mới dữ liệu thành công!");
            ReloadData();
            ResetForm();
        }

        private void btnEdit_Click(object sender, EventArgs e)
        {
            using (Model1 context = new Model1())
            {
                var db = context.Students.FirstOrDefault(p => p.StudentID == txtStudentID.Text);
                if (db == null)
                {
                    MessageBox.Show("Không tìm thấy MSSV cần sửa!");
                    return;
                }

                db.FullName = txtFullName.Text;
                db.AverageScore = double.Parse(txtAverageScore.Text);
                db.FacultyID = cmbFaculty.SelectedValue.ToString();

                context.SaveChanges();
            }

            MessageBox.Show("Cập nhật dữ liệu thành công!");
            ReloadData();
            ResetForm();
        }

     
        private void btnDelete_Click(object sender, EventArgs e)
        {
            using (Model1 context = new Model1())
            {
                var db = context.Students.FirstOrDefault(p => p.StudentID == txtStudentID.Text);
                if (db == null)
                {
                    MessageBox.Show("Không tìm thấy MSSV cần xóa!");
                    return;
                }

                DialogResult dr = MessageBox.Show("Bạn có chắc muốn xóa sinh viên này?",
                                                  "Xác nhận", MessageBoxButtons.YesNo, MessageBoxIcon.Question);
                if (dr == DialogResult.Yes)
                {
                    context.Students.Remove(db);
                    context.SaveChanges();
                    MessageBox.Show("Xóa sinh viên thành công!");
                    ReloadData();
                    ResetForm();
                }
            }
        }


        private void dgvStudent_CellClick(object sender, DataGridViewCellEventArgs e)
        {
            if (e.RowIndex >= 0)
            {
                DataGridViewRow row = dgvStudent.Rows[e.RowIndex];
                txtStudentID.Text = row.Cells[0].Value?.ToString();
                txtFullName.Text = row.Cells[1].Value?.ToString();
                cmbFaculty.Text = row.Cells[2].Value?.ToString();
                txtAverageScore.Text = row.Cells[3].Value?.ToString();
            }
        }

   
        private void ReloadData()
        {
            Model1 context = new Model1();
            List<Student> listStudents = context.Students.ToList();
            BindGrid(listStudents);
        }

        
        private void ResetForm()
        {
            txtStudentID.Clear();
            txtFullName.Clear();
            txtAverageScore.Clear();
            cmbFaculty.SelectedIndex = 0;
        }

        private void btnExit_Click(object sender, EventArgs e)
        {
            this.Close();
        }
    }
}
