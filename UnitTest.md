# Unit Tests cho lớp `People` trong namespace `HomeWork.Way1`

Dưới đây là các bài kiểm thử (unit tests) được viết bằng `XUnit` để kiểm tra chức năng liên quan đến quan hệ huyết thống và điều kiện kết hôn giữa các đối tượng `People`.

```csharp
using HomeWork.Way1;
using Xunit;

namespace TestProject
{
    public class PeopleMarriageTests
    {
        /// <summary>
        /// Tạo một cây gia phả mẫu để test:
        /// Cụ (GGP) -> Ông (GP) -> Cha (P1) & Cô (P2) -> Con1 (C1) & Con2 (C2)
        private (People C1, People C2, People P1, People P2, People GP, People GGP) SetupFamilyTree()
        {
            var ggp = new People("Cụ Nội", "Nam");
            var gp = new People("Ông Nội", "Nam");
            ggp.AddChild(gp);

            var p1 = new People("Cha", "Nam");
            var p2 = new People("Cô", "Nữ");
            gp.AddChild(p1);
            gp.AddChild(p2);

            var c1 = new People("Con Trai (Của Cha)", "Nam");
            var c2 = new People("Con Gái (Của Cô)", "Nữ");
            p1.AddChild(c1);
            p2.AddChild(c2);

            return (c1, c2, p1, p2, gp, ggp);
        }

        [Fact]
        public void CanMarry_SamePerson_ReturnsFalse()
        {
            // Arrange
            var p = new People("An", "Nam");

            // Act
            bool result = p.CanMarry(p);

            // Assert
            Assert.False(result); // Không thể tự cưới chính mình
        }

        [Fact]
        public void CanMarry_SameGender_ReturnsFalse()
        {
            // Arrange
            var p1 = new People("An", "Nam");
            var p2 = new People("Bình", "Nam");

            // Act
            bool result = p1.CanMarry(p2);

            // Assert
            Assert.False(result); // Cùng giới tính trả về false theo logic code
        }

        [Fact]
        public void CanMarry_ParentAndChild_ReturnsFalse()
        {
            // Arrange
            var parent = new People("Cha", "Nam");
            var child = new People("Con", "Nữ");
            parent.AddChild(child);

            // Act & Assert
            Assert.False(parent.CanMarry(child)); // Huyết thống trực hệ
            Assert.False(child.CanMarry(parent));
        }

        [Fact]
        public void CanMarry_Siblings_ReturnsFalse()
        {
            // Arrange
            var parent = new People("Bố", "Nam");
            var c1 = new People("Anh", "Nam");
            var c2 = new People("Em Gái", "Nữ");
            parent.AddChild(c1);
            parent.AddChild(c2);

            // Act
            bool result = c1.CanMarry(c2);

            // Assert
            Assert.False(result); // Anh em ruột (Đời thứ 2 tính từ tổ tiên chung là Ông)
        }

        [Fact]
        public void CanMarry_FirstCousins_ReturnsFalse()
        {
            // Arrange
            var family = SetupFamilyTree();
            // C1 và C2 là anh em họ (Chung ông nội GP)

            // Act
            bool result = family.C1.CanMarry(family.C2);

            // Assert
            // Đời: C1(3) - P1(2) - GP(1)
            // Đời: C2(3) - P2(2) - GP(1)
            // Cùng <= 3 đời nên phải trả về False
            Assert.False(result);
        }

        [Fact]
        public void IsDirectBloodline_GrandParentAndGrandChild_ReturnsTrue()
        {
            // Arrange
            var family = SetupFamilyTree();

            // Act
            bool isDirect = family.GP.IsDirectBloodline(family.C1);

            // Assert
            Assert.True(isDirect); // Ông và Cháu là trực hệ
        }

        [Fact]
        public void GetLowestCommonAncestor_ReturnsCorrectAncestor()
        {
            // Arrange
            var family = SetupFamilyTree();

            // Act
            var lca = family.C1.GetLowestCommonAncestor(family.C2);

            // Assert
            Assert.NotNull(lca);
            Assert.Equal(family.GP.Id, lca.Id); // Tổ tiên chung gần nhất của anh em họ là Ông Nội
        }

        [Fact]
        public void CanMarry_Strangers_ReturnsTrue()
        {
            // Arrange
            var p1 = new People("Người Lạ 1", "Nam");
            var p2 = new People("Người Lạ 2", "Nữ");

            // Act
            bool result = p1.CanMarry(p2);

            // Assert
            Assert.True(result); // Không có tổ tiên chung, khác giới tính -> Cưới được
        }

        [Fact]
        public void GetGenerationRelativeTo_ReturnsCorrectDistance()
        {
            // Arrange
            var family = SetupFamilyTree();

            // Act
            int distance = family.C1.GetGenerationRelativeTo(family.GP);

            // Assert
            // C1 là đời 1, P1 là đời 2, GP là đời 3
            Assert.Equal(3, distance);
        }
    }
}
