
���˵������ tag �������ύ, ��ô branch ���� �����ύ��. 
branch ָ���ύ����ͷ�ڵ�, �ύ�����ӽڵ�ʱ, branch �Զ�ǰ��ָ���µ�ͷ�ڵ�.

������ tag �������Ը��ύ����, �����Ը�һ�� object ����, �� tree, blob, tag.

��֧���� tag ��ֻ�Ƕ�������, ����˵��������.
��ͬ���͵�������Ϊ��ͬ, ���� tag �� branch �����ύ���������, 
����;����Ϊ��һ��.


## git �� svn ����
1. ����ṹ, �ֲ�ʽ, �汸�ͻ�����������Ĺ���, �����ֿͻ����������
2. �����ķ�֧����������֧��ֻ��һ������. ��֧�ϲ�Ч�����޸ļ�¼����ʧ����κϲ�����ͻ��


## ��ѯ��ͬ��Զ�ֿ̲�

### ���ٺ͸���

* git push
push.default Ӱ����Ϊ, 1.x �����޸�Ϊ upstream, 2.x Ĭ�� simple
* git pull
* git branch --set-upstream local-branch remote-branch

### git remote 

* ����Զ����Ϣ������
git remote update
git remote update origin
git remote update apache70

* �鿴Զ�ֿ̲�״̬, �Աȱ��زֿ�
git remote show 

### git ls-remote

�鿴Զ�ֿ̲��ϵ�����(branch, tag)�б�

git ls-remote
git ls-remote apache70

