```csharp
namespace HomeWork.Way1
{
    public class People
    {
        public string Id { get; set; } = Guid.NewGuid().ToString();
        public string Gender { get; set; }
        public string Name { get; set; }
        public People? Parent { get; set; }
        public List<People> Children { get; set; } = new List<People>();
        private const int MAX_SEARCH_DEPTH = 4; // Thiết lập độ sâu tối đa ưu tiên chỉ kiểm tra 3 đời có được cưới hay không

        public People(string name, string gender)
        {
            Name = name;
            Gender = gender;
        }

        public void AddChild(People child)
        {
            child.Parent = this;
            Children.Add(child);
        }

        public bool IsSameParent(People other)
        {
            if (Parent == null || other.Parent == null) return false;
            return Parent == other.Parent;
        }

        public bool IsSameGrandParent(People other)
        {
            if (Parent?.Parent == null || other.Parent?.Parent == null) return false;
            return Parent.Parent == other.Parent.Parent;
        }

        public bool IsSameGreatGrandParent(People other)
        {
            if (Parent?.Parent?.Parent == null || other.Parent?.Parent?.Parent == null) return false;
            return Parent.Parent.Parent == other.Parent.Parent.Parent;
        }

        // 1. Kiểm tra có phải huyết thống trực hệ không 
        public bool IsDirectBloodline(People other)
        {
            return IsAncestorOf(other) || other.IsAncestorOf(this);
        }

        // Kiểm tra xem mình có phải tổ tiên của người kia không
        public bool IsAncestorOf(People other)
        {
            People? current = other.Parent;
            int deepth = 1;
            while (current != null && deepth != MAX_SEARCH_DEPTH)
            {
                if (current == this) return true;
                current = current.Parent;
                deepth++;
            }
            return false;
        }

        // 2. Tìm Tổ tiên chung gần nhất (Người có vai vế lớn hơn gần nhất) 
        public People? GetLowestCommonAncestor(People other)
        {
            int deepth = 0;
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
            while (current != null && deepth != MAX_SEARCH_DEPTH)
            {
                if (myAncestors.Contains(current)) return current;
                current = current.Parent;
                deepth++;
            }
            return null;
        }

        // Xác định đời so với tổ tiên chung
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

        //Kiểm tra có phải cùng một người không
        public bool IsMySelf(People other)
        {
            if (Id == other.Id) return true;
            return false;
        }

        // Kiểm tra giới tính hợp lệ để cưới
        public bool IsNotGenderValid(People other)
        {
            if (Gender == other.Gender) return true;
            return false;
        }

        // Kiểm tra có nằm trong 3 đời cấm kết hôn không
        public bool IsWithinForbiddenGenerations(People other, People commonAncestor)
        {
            int myGen = GetGenerationRelativeTo(commonAncestor);
            int otherGen = other.GetGenerationRelativeTo(commonAncestor);

            if (myGen <= 3 && otherGen <= 3)
            {
                return true;
            }
            return false;
        }

        // Kiểm tra có tổ tiên chung nào không
        public bool HasNotLowestCommonAncestor(People? ancestor)
        {
            if (ancestor == null) return true;
            return false;
        }

        public bool CanMarry(People other)
        {
            // Nếu là cùng một người thì không được phép kết hôn
            if (IsMySelf(other)) return false;
            // Nếu cùng giới tính thì không được phép kết hôn
            if (IsNotGenderValid(other)) return false;
            // Nếu là huyết thống trực hệ thì không được phép kết hôn
            if (IsDirectBloodline(other)) return false;
            People? commonAncestor = GetLowestCommonAncestor(other);
            // Nếu không có tổ tiên chung (người có vai vế lớn hơn gần nhất) nào thì có thể kết hôn
            if (HasNotLowestCommonAncestor(commonAncestor)) return true;
            // Nếu có tổ tiên chung (người có vai vế lớn hơn gần nhất) nhưng không nằm trong 3 đời thì có thể kết hôn
            if (IsWithinForbiddenGenerations(other, commonAncestor)) return false;
            return true;

        }
    }
}
