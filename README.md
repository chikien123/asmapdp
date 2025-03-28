using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using SIMS.Models;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication;
using System.Security.Claims;
using Microsoft.AspNetCore.Authorization;

namespace SIMS.Controllers
{
    public class AdminsController : Controller
    {
        private readonly SimsContext _context;

        public AdminsController(SimsContext context)
        {
            _context = context;
        }
        [HttpGet]
        public IActionResult Login()
        {
            return View();
        }
        [Authorize(Roles = "Admin")]

        public IActionResult DashBoard()
        {
            var studentCount = _context.Students.Count();
            var classCount = _context.Classes.Count();
            var courseCount = _context.Courses.Count();
            var scoreCount = _context.Scores.Count(); // Thêm dòng này để lấy số lượng điểm
            ViewBag.StudentCount = studentCount;
            ViewBag.ClassCount = classCount;
            ViewBag.CourseCount = courseCount;
            ViewBag.ScoreCount = scoreCount;

            return View();
        }




        [HttpPost]
        public async Task<IActionResult> Login(Admin model)
        {
            if (ModelState.IsValid)
            {
                var admin = _context.Admins.FirstOrDefault(a => a.UserName == model.UserName && a.Password == model.Password);

                if (admin == null)
                {
                    ViewBag.ErrorMessage = "Tai khoan khong ton tai";
                    return View();
                }
                else
                {
                    var role = _context.Roles.FirstOrDefault(r => r.RoleId == admin.RoleId);
                    var claims = new List<Claim>
                    {
                        new Claim(ClaimTypes.NameIdentifier, admin.AdminId.ToString()),
                        new Claim(ClaimTypes.Role, "Admin"),
                        //new Claim("Email", student.Email)
                    };
                    var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
                    var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);

                    await HttpContext.SignInAsync(claimsPrincipal);

                    return RedirectToAction("DashBoard", "Admins");
                }
            }
            return View();
        }

   
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> CreateStudent(Student model)
        {

                var existingST = _context.Students.FirstOrDefault(a => a.UserName == model.UserName);

                if (existingST != null)
                {
                    ViewBag.ErrorMessage = "Tai khoan da ton tai";
                    return View();
                }

                // Create new student and add to context
                var student = new Student
                {
                    UserName = model.UserName,
                    Password = model.Password,
                    FullName = model.FullName,
                    Email = model.Email,    
                    Address = model.Address,
                    DateOfBirth = model.DateOfBirth,
                    RoleId = model.RoleId,
                    
                    // Add any other required fields
                };
                
                _context.Students.Add(student);
                _context.SaveChangesAsync();

                return RedirectToAction("Index", "Students");
            
            return View();
        }

        [Authorize(Roles = "Admin")]
        public IActionResult CreateStudent()
        {
            return View();
        }

        public async Task <IActionResult> logout()
        {
            await HttpContext.SignOutAsync();
            return RedirectToAction("Login", "Admins");
          
        }

    }
}
