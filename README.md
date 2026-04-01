# Lớp `People` - Quản lý quan hệ huyết thống và điều kiện kết hôn

Dưới đây là toàn bộ code lớp `People` trong namespace `HomeWork`.  
Bạn có thể xem xét trực tiếp logic và các hàm liên quan đến quản lý quan hệ gia đình, tìm tổ tiên chung và kiểm tra điều kiện kết hôn hợp lệ.

```csharp
namespace HomeWork
{
    public class People
    {
        public string Gender { get; set; }
        public string Name { get; set; }
        public People? Parent { get; set; }
        public List<People> Children { get; set; } = new List<People>();

        public People(string name)
        {
            Name = name;
        }

        public void AddChild(People child)
        {
            child.Parent = this;
            Children.Add(child);
        }

        public bool IsSameParent(People other)
        {
            if (this.Parent == null || other.Parent == null) return false;
            return this.Parent == other.Parent;
        }

        public bool IsSameGrandParent(People other)
        {
            if (this.Parent?.Parent == null || other.Parent?.Parent == null) return false;
            return this.Parent.Parent == other.Parent.Parent;
        }

        public bool IsSameGreatGrandParent(People other)
        {
            if (this.Parent?.Parent?.Parent == null || other.Parent?.Parent?.Parent == null) return false;
            return this.Parent.Parent.Parent == other.Parent.Parent.Parent;
        }

        // 1. Kiểm tra có phải huyết thống trực hệ không (Cha mẹ - con cái, ông bà - cháu...)
        public bool IsDirectBloodline(People other)
        {
            return this.IsAncestorOf(other) || other.IsAncestorOf(this);
        }

        // Kiểm tra xem mình có phải tổ tiên của người kia không
        public bool IsAncestorOf(People other)
        {
            People? current = other.Parent;
            while (current != null)
            {
                if (current == this) return true;
                current = current.Parent;
            }
            return false;
        }

        // 2. Tìm Tổ tiên chung gần nhất (Lowest Common Ancestor)
        public People? GetCommonAncestor(People other)
        {
            var myAncestors = new HashSet<People>();
            People? current = this;

            // Lưu lại toàn bộ gốc gác của mình (bao gồm cả bản thân)
            while (current != null)
            {
                myAncestors.Add(current);
                current = current.Parent;
            }

            // Duyệt từ người kia lên, ai trùng đầu tiên thì đó là tổ tiên chung gần nhất
            current = other;
            while (current != null)
            {
                if (myAncestors.Contains(current))
                {
                    return current;
                }
                current = current.Parent;
            }

            return null;
        }

        public int GetGenerationRelativeTo(People ancestor)
        {
            int distance = 1; // Tổ tiên chung tính là đời thứ 1
            People? current = this;
            while (current != null)
            {
                if (current == ancestor) return distance;
                distance++;
                current = current.Parent;
            }
            return -1; // Không cùng dòng họ với người này
        }

        // 4. Kiểm tra điều kiện kết hôn
        public bool CanMarry(People other)
        {
            if (this == other) return false;

            if (this.Gender == other.Gender) return false;

            // Không được cưới người trực hệ (cha mẹ, ông bà, con cháu...)
            if (IsDirectBloodline(other)) return false;

            // Tìm tổ tiên chung
            People? commonAncestor = GetCommonAncestor(other);

            // Nếu không có tổ tiên chung -> Không cùng huyết thống -> Được cưới
            if (commonAncestor == null) return true;

            // Tính số đời của cả 2 so với tổ tiên chung
            int myGen = this.GetGenerationRelativeTo(commonAncestor);
            int otherGen = other.GetGenerationRelativeTo(commonAncestor);

            // Cấm kết hôn nếu cả 2 người đều thuộc phạm vi 3 đời
            if (myGen <= 3 && otherGen <= 3)
            {
                return false;
            }

            return true;
        }

        public string GetRelationshipInfo(People other)
        {
            if (this == other) return $"{this.Name} là chính mình.";
            if (IsDirectBloodline(other)) return $"{this.Name} và {other.Name} là họ hàng trực hệ.";

            People? commonAncestor = GetCommonAncestor(other);
            if (commonAncestor == null) return $"{this.Name} và {other.Name} không có quan hệ huyết thống.";

            int myGen = GetGenerationRelativeTo(commonAncestor);
            int otherGen = other.GetGenerationRelativeTo(commonAncestor);

            string status = CanMarry(other) ? "ĐƯỢC PHÉP cưới" : "KHÔNG ĐƯỢC cưới";
            return $"Tổ tiên chung là: {commonAncestor.Name}. {this.Name} (Đời {myGen}) - {other.Name} (Đời {otherGen}) -> {status}.";
        }
    }
}
